---
layout: post
title:  KASAN Info Leak Detection 
categories: [Apple,XNU]
---

In my previous blog post I dug into a general overview of the [KASAN implementation in XNU](http://127.0.0.1:4000/macos-kasan/). This post goes more in depth in detecting kernel uninitialized information leaks using it (no 0days dropped here :)). Kernel Information Leaks to userland are a common problem and can be used to leak confidential information or disclose kernel memory addresses which are typically used to bypass KASLR. An example of this type of vulnerability is [CVE-2017-13868](https://bazad.github.io/2018/03/a-fun-xnu-infoleak/). 

We can see within the XNU KASAN sources that attempts have been made to detect these types of issues when running the instrumented kernel build.

When memory is allocated using the ```kasan_alloc``` function, a requested size of memory is memset with a heap fill pattern of 0xbe: 

```c
/* uninitialized memory detection */
#define KASAN_UNINITIALIZED_HEAP   0xbe

vm_address_t
kasan_alloc(vm_offset_t addr, vm_size_t size, vm_size_t req, vm_size_t leftrz)
{
	if (!addr) {
		return 0;
	}
	assert(size > 0);
	assert((addr % 8) == 0);
	assert((size % 8) == 0);

	vm_size_t rightrz = size - req - leftrz;

	kasan_poison(addr, req, leftrz, rightrz, ASAN_HEAP_RZ);
	kasan_rz_clobber(addr, req, leftrz, rightrz);

	addr += leftrz;

	if (enabled_checks & TYPE_LEAK) {
		__nosan_memset((void *)addr, KASAN_UNINITIALIZED_HEAP, req);
	}
```

A check is then performed when performing copying out data from kernel space to userspace (using the ```copyout``` function), ```copy_validate``` is called:

```c
static int
copy_validate(const user_addr_t user_addr, uintptr_t kernel_addr,
    vm_size_t nbytes, copyio_flags_t flags)
{
	...

#if KASAN
		/* For user copies, asan-check the kernel-side buffer */
		if (flags & COPYIO_IN) {
			__asan_storeN(kernel_addr, nbytes);
		} else {
			__asan_loadN(kernel_addr, nbytes);
			kasan_check_uninitialized((vm_address_t)kernel_addr, nbytes);
		}
#endif
```
and the ```copyio``` code has been modified to also introduce a check:

```c
#if KASAN
	switch (copy_type) {
	case COPYIN:
	case COPYINSTR:
	case COPYINATOMIC32:
	case COPYINATOMIC64:
		__asan_storeN((uptr)kernel_addr, nbytes);
		break;
	case COPYOUT:
	case COPYOUTATOMIC32:
	case COPYOUTATOMIC64:
		__asan_loadN((uptr)kernel_addr, nbytes);
		kasan_check_uninitialized((vm_address_t)kernel_addr, nbytes);
		break;
	}
#endif
```

The check determines if the heap fill pattern is included within the data which is going to be copied to userspace. If the max_count of leaked bytes is >= than the leak_threshold, the leak is reported:  

```c

/*
 * Check for possible uninitialized memory contained in [base, base+sz).
 */
void
kasan_check_uninitialized(vm_address_t base, vm_size_t sz)
{
	if (!(enabled_checks & TYPE_LEAK) || sz < leak_threshold) {
		return;
	}

	vm_address_t cur = base;
	vm_address_t end = base + sz;
	vm_size_t count = 0;
	vm_size_t max_count = 0;
	vm_address_t leak_offset = 0;
	uint8_t byte = 0;

	while (cur < end) {
		byte = *(uint8_t *)cur;
		count = (byte == KASAN_UNINITIALIZED_HEAP) ? (count + 1) : 0;
		if (count > max_count) {
			max_count = count;
			leak_offset = cur - (count - 1) - base;
		}
		cur += 1;
	}

	if (max_count >= leak_threshold) {
		kasan_report_leak(base, sz, leak_offset, max_count);
	}
}
```

So how do we enable this feature and get hold of the output? 

```KASAN_ARGS_CHECK_LEAKS 0x0800U``` can be used to enable the feature:

```bash
sudo nvram boot-args="-v keepsyms=1 debug=0x2444 kasan=0x0800 kcsuffix=kasan"
```
Then we need find out how to obtain the output: 

```c
static void
kasan_report_leak(vm_address_t base, vm_size_t sz, vm_offset_t offset, vm_size_t leak_sz)
{
	if (leak_fatal_threshold > leak_threshold && leak_sz >= leak_fatal_threshold){
		kasan_violation(base + offset, leak_sz, TYPE_LEAK, REASON_UNINITIALIZED);
	}

	char string_rep[BACKTRACE_MAXFRAMES * 20] = {};
	vm_offset_t stack_base = dtrace_get_kernel_stack(current_thread());
	bool is_stack = (base >= stack_base && base < (stack_base + kernel_stack_size));

	if (!is_stack) {
		uintptr_t alloc_bt[BACKTRACE_MAXFRAMES] = {};
		vm_size_t num_frames = 0;
		size_t l = 0;
		num_frames = kasan_alloc_retrieve_bt(base, alloc_bt);
		for (vm_size_t i = 0; i < num_frames; i++) {
			l += snprintf(string_rep + l, sizeof(string_rep) - l, " %lx", alloc_bt[i]);
		}
	}

	DTRACE_KASAN5(leak_detected,
				  vm_address_t, base,      
				  vm_size_t, sz,           
				  vm_offset_t, offset,     
				  vm_size_t, leak_sz,      
				  char *, string_rep);    
}
```

Looking at this we can see there are two methods:

* Turning leaks into a fatal crash

* Using dtrace to log the leak  

The downside of the first method is that it is way harder to debug and root cause the leak with post mortem debugging compared to a live running kernel. Therefore, I explored more of the [dtrace](http://dtrace.org/blogs/about/) method. For those of you not familiar, dtrace is a dynamic tracing framework typically used for kernel and application troubleshooting. There has been a number of great security relevant presentations in the past about using dtrace to aid [reverse engineering](https://www.blackhat.com/presentations/bh-usa-08/Beauchamp_Weston/BH_US_08_Beauchamp-Weston_DTrace.pdf) or [bugs within the dtrace subsystem](https://securitylab.github.com/research/apple-xnu-dtrace-CVE-2017-13782) itself.    

In order to obtain the output, we can make use of the dtrace probe registered by KASAN. You can see this by listing the dtrace probes or looking in the source:

```bash
ID   PROVIDER            MODULE                          FUNCTION NAME
.. 
1840      kasan       mach_kernel         kasan_check_uninitialized leak_detected
```

Then we can write a dtrace script as follows, which will be used to log the leak address, offset and size. I also use the [tracemem](https://docs.oracle.com/cd/E19253-01/819-5488/gcgge/index.html) function to provide a nice hexdump of the memory: 

```c
kasan::kasan_check_uninitialized:leak_detected
{
    printf ("kasan leak at 0x%p of size %u, offset %u leak size %u repr: %s ",arg0,arg1,arg2,arg3,stringof(arg4));
    tracemem(arg0, 512);
}
```
This can be run using ```dtrace -s kasan.d```

By default the leak threshold is set to => 3 bytes. This can also be controlled by both an nvram or systl setting (leak_threshold=whatever).  

In the case of the address being on the kernel stack, a string representation is created of where it was initially allocated. However, throughout testing, it was determined that the stack trace was not symbolized, and therefore required jumping into a kernel debugger to determine the symbolic location. 

Ideally we would like the stack of where the copyout is called from too. To do this we can simple add a [stack](https://docs.oracle.com/cd/E18752_01/html/819-5488/gcfbn.html#gcgfo) call to our dtrace script:

```c
kasan::kasan_check_uninitialized:leak_detected
{
    printf ("kasan leak at 0x%p of size %u, offset %u leak size %u repr: %s ",arg0,arg1,arg2,arg3,stringof(arg4));
    tracemem(arg0, 512);
    stack()
}
```

Since we are looking for uninitialized memory bugs which are easily reproducable, what we could also do would be to modify any output received by a fuzzer to detect the fill pattern (0xbe). For example, a common location would be [IOConnectCallMethod](https://developer.apple.com/documentation/iokit/1514240-ioconnectcallmethod?language=objc) output and outputStruct values. An example of such a leak is [CVE-2015-5864 - Heap Info Leak](https://github.com/jndok/tpwn-bis/blob/cb7760c587d7080545fc98d0a4d42b802f5de62e/poc-1/pwn.m#L25) where a kernel address was being leaked to userspace.   

The downside with this approach is that you really need to cover all the possible wrappers which use ```copyout```. Therefore using both the dtrace technique and some manual effort to locate the right areas to audit, we can discover when uninitialized data is being copied into userspace. Then we can drop to kernel debugging to confirm the issue.  