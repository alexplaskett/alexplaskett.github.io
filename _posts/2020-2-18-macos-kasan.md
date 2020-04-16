---
layout: post
title:  A brief introduction into KASAN on macOS Catalina
categories: [Apple,XNU]
---

This article will show some initial research into booting a KSAN kernel, testing the KASAN functionality and some initial groundwork on KSANCOV. This functionality is super useful when performing kernel crash triage or fuzzing against macOS.  

KSAN is Apple's implementation of [AddressSanitizer](https://github.com/google/sanitizers/wiki/AddressSanitizer) within the kernel and is used to detect memory errors. 

Apple has recently published "Kernel Debug Kit 10.15.4 build 19E287", which is actually the KDK for the latest production macOS version (10.15.4 Supplemental Updates) at the time of writing. In the past the KDK versions have often lagged to the current version or have only been available for beta builds. These build versions provide a good base for investigation into KASAN without having symbol issues to deal with due to mixed versions.   

~~At the time of writing the dependancies for 10.15 have not been published on [Apple Opensource](https://opensource.apple.com/release/macos-1015.html) preventing building a KSAN kernel from source (EDIT: more sources were added yesterday so this might now be buildable!). However, a number of KDK builds have been published which include KASAN support. Unfortunately, there are no KDK builds for the stable release version of the OS. However, a developer build version kernel can be booted on the right version of a production macOS with a bit of hackery. For example, macOS 10.15.2 (19C57) can use KDK_10.15.2_19C39d.kdk kernel and successfully boot with no dependancy or symbol issues.~~ 

Within the KDK (/Library/Developer/KDKs/KDK_10.15.4_19E287.kdk/System/Library/Kernels/) we have the following versions of the kernel: 
* kernel (release build)
* kernel.debug (debug)
* kernel.development (development)
* kernel.kasan (kasan)

As a quick check before deployment we can compare the existing production kernel hash against the release kernel in the KDK (and see they match):
```
3734120155ff70c7a05c1b46d26cc1622a6c46ff  /Library/Developer/KDKs/KDK_10.15.4_19E287.kdk/System/Library/Kernels/kernel
3734120155ff70c7a05c1b46d26cc1622a6c46ff  /System/Library/Kernels/kernel
``` 

Now we can deploy both the kernels and IOKit drivers to the guest VM. After we have installed the KDK package onto the VM. The process for deploying and selecting the kasan kernel to boot is as follows (this requires SIP disabled):

```bash
sudo mount -uw /
sudo cp /Library/Developer/KDKs/KDK_10.15.4_19E287.kdk/System/Library/Kernels/kernel.kasan /System/Library/Kernels/
sudo cp -r /Library/Developer/KDKs/KDK_10.15.4_19E287.kdk/System/Library/Extensions/ /System/Library/Extensions/
sudo kextcache -invalidate /
sudo nvram boot-args="-v keepsyms=1 debug=0x2444 kasan.checks=24576 -zp -zc kcsuffix=kasan"
```

After rebooting into the kernel we should see we are running within the KASAN kernel (in uname -a output):

```bash
Darwin Mac.local 19.4.0 Darwin Kernel Version 19.4.0: Wed Mar  4 22:30:14 PST 2020; root:xnu_kasan-6153.101.6~12/KASAN_X86_64 x86_64
```
It should be noted that within the KDK there are also IOKit kernel extensions compiled with KSAN. This can be very useful if analyzing a bug within a kernel extension which is provided. 

To check that a KEXT is loaded with KASAN we can examined the UUID and check they match:

```bash
$ sudo kextstat | grep IOHID
   53    3 0xffffff7f820eb000 0x1e4000   0x1e4000   com.apple.iokit.IOHIDFamily (2.0.0) 9DAEAA3D-2F24-31D1-9065-EB5871936466 <17 8 7 6 5 3 2 1>
$ dwarfdump -u /Library/Developer/KDKs/KDK_10.15.4_19E287.kdk/System/Library/Extensions/IOHIDFamily.kext/Contents/MacOS/IOHIDFamily_kasan
UUID: 9DAEAA3D-2F24-31D1-9065-EB5871936466 (x86_64) /Library/Developer/KDKs/KDK_10.15.4_19E287.kdk/System/Library/Extensions/IOHIDFamily.kext/Contents/MacOS/IOHIDFamily_kasan
```

And we can check that KASAN is loaded correctly by examining 'sysctl kern.kasan.available' which should be 1 if running under KASAN.

There are a number of other sysctl's which describe kasan's operation and can be tweaked:

```bash
boxname:~ user$ sysctl -a | grep kasan
kern.version: Darwin Kernel Version 19.4.0: Wed Mar  4 22:30:14 PST 2020; root:xnu_kasan-6153.101.6~12/KASAN_X86_64
kern.bootargs: -v keepsyms=1 debug=0x2444 -zp -zc kcsuffix=kasan
kern.kasan.available: 1
kern.kasan.enabled: 1
kern.kasan.checks: 4294901759
kern.kasan.quarantine: 1
kern.kasan.report_ignored: 0
kern.kasan.free_yield_ms: 0
kern.kasan.leak_threshold: 3
kern.kasan.leak_fatal_threshold: 0
kern.kasan.memused: 22871
kern.kasan.memtotal: 131074
kern.kasan.kexts: 15
kern.kasan.debug: 0
kern.kasan.zalloc: 1
kern.kasan.kalloc: 1
kern.kasan.dynamicbl: 1
kern.kasan.fakestack: 0
kern.kasan.test: 0
kern.kasan.fail: 0
```
We can test KASAN is functioning like so (Test double free):

```bash
sudo sysctl -w kern.kasan.test=100
```
We should then see the following crash:

```bash
Process 1 stopped
* thread #1, stop reason = signal SIGSTOP
    frame #0: 0xffffff801f0aae7e kernel.kasan`DebuggerTrapWithState + 78
kernel.kasan`DebuggerTrapWithState:
->  0xffffff801f0aae7e <+78>: callq  0xffffff801f3fa7a0        ; current_processor
    0xffffff801f0aae83 <+83>: movl   0x5c0(%rax), %ebx
    0xffffff801f0aae89 <+89>: xorl   %edi, %edi
    0xffffff801f0aae8b <+91>: xorl   %esi, %esi
Target 0: (kernel.kasan) stopped.
(lldb) bt
* thread #1, stop reason = signal SIGSTOP
  * frame #0: 0xffffff801f0aae7e kernel.kasan`DebuggerTrapWithState + 78
    frame #1: 0xffffff8020710756 kernel.kasan`panic_trap_to_debugger.cold.1 + 166
    frame #2: 0xffffff801f0aba82 kernel.kasan`panic_trap_to_debugger + 338
    frame #3: 0xffffff8020710392 kernel.kasan`panic + 98
    frame #4: 0xffffff8020720209 kernel.kasan`kasan_report_internal.cold.1 + 25
    frame #5: 0xffffff8020705694 kernel.kasan`kasan_report_internal + 820
    frame #6: 0xffffff8020703233 kernel.kasan`kasan_crash_report + 51
    frame #7: 0xffffff8020702ce1 kernel.kasan`kasan_violation + 673
    frame #8: 0xffffff8020703f4f kernel.kasan`kasan_check_free + 207
    frame #9: 0xffffff801f0ca459 kernel.kasan`kfree + 169
    frame #10: 0xffffff8020706c69 kernel.kasan`heap_cleanup + 89
    frame #11: 0xffffff80207068bd kernel.kasan`kasan_run_test + 429
    frame #12: 0xffffff80207066c7 kernel.kasan`kasan_test + 71
    frame #13: 0xffffff802070534b kernel.kasan`sysctl_kasan_test + 75
    frame #14: 0xffffff8020064590 kernel.kasan`sysctl_root + 1904
    frame #15: 0xffffff8020064dc5 kernel.kasan`sysctl + 1285
    frame #16: 0xffffff80203bcf00 kernel.kasan`unix_syscall64 + 2192
    frame #17: 0xffffff801f44be26 kernel.kasan`hndl_unix_scall64 + 22

panic(cpu 1 caller 0xffffff8020720209): "KASan: free of corrupted/invalid object 0xffffff802d3cd880
 Shadow             0  1  2  3  4  5  6  7  8  9  a  b  c  d  e  f
 fffff7f005a79ac0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79ad0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79ae0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79af0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79b00: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
 fffff7f005a79b10:[00]00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79b20: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79b30: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79b40: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79b50: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
 fffff7f005a79b60: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 
```

Looking through the sources for 10.15 (san/kasan-test.c) we can see tests for the following:
* Global Overflows
* Heap Underflows
* Heap Overflows
* Heap Use-After-Frees
* Heap Invalid Frees
* Heap Double Frees
* Heap Small Frees
* Stack Overflows
* Stack Underflows
* Stack Use-After-Returns
* Specific operations which could read/write OOB (memcpy, memmove, bcopy, memset, bcmp, bzero, strlcpy, strlcat, strncat)

The KASAN checks can be enabled or disabled by setting the nvram boot-args variable to contain 'kasan.checks', this is bitmask as follows:

```c
static unsigned enabled_checks = TYPE_ALL & ~TYPE_LEAK; /* bitmask of enabled checks */

enum __attribute__((flag_enum)) kasan_access_types {
	TYPE_LOAD    = BIT(0),  /* regular memory load */
	TYPE_STORE   = BIT(1),  /* regular store */
	TYPE_MEMR    = BIT(2),  /* memory intrinsic (read) */
	TYPE_MEMW    = BIT(3),  /* memory intrinsic (write) */
	TYPE_STRR    = BIT(4),  /* string intrinsic (read) */
	TYPE_STRW    = BIT(5),  /* string intrinsic (write) */
	TYPE_KFREE   = BIT(6),  /* kfree() */
	TYPE_ZFREE   = BIT(7),  /* zfree() */
	TYPE_FSFREE  = BIT(8),  /* fakestack free */

	TYPE_UAF           = BIT(12),
	TYPE_POISON_GLOBAL = BIT(13),
	TYPE_POISON_HEAP   = BIT(14),
	/* no TYPE_POISON_STACK, because the runtime does not control stack poisoning */
	TYPE_TEST          = BIT(15),
	TYPE_LEAK          = BIT(16),

	/* masks */
	TYPE_MEM     = TYPE_MEMR | TYPE_MEMW,            /* memory intrinsics */
	TYPE_STR     = TYPE_STRR | TYPE_STRW,            /* string intrinsics */
	TYPE_READ    = TYPE_LOAD | TYPE_MEMR | TYPE_STRR,  /* all reads */
	TYPE_WRITE   = TYPE_STORE | TYPE_MEMW | TYPE_STRW, /* all writes */
	TYPE_RW      = TYPE_READ | TYPE_WRITE,           /* reads and writes */
	TYPE_FREE    = TYPE_KFREE | TYPE_ZFREE | TYPE_FSFREE,
	TYPE_NORMAL  = TYPE_RW | TYPE_FREE,
	TYPE_DYNAMIC = TYPE_NORMAL | TYPE_UAF,
	TYPE_POISON  = TYPE_POISON_GLOBAL | TYPE_POISON_HEAP,
	TYPE_ALL     = ~0U,
};
```

One interesting recent introduction is the detection of uninitialized memory on the heap:
```c
/* uninitialized memory detection */
#define KASAN_UNINITIALIZED_HEAP   0xbe

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

There is also a KSAN dynamic blacklist which can be used at compile time or a dynamic one using nvram flag 'kasan.bl':
```c
void
kasan_init_dybl(void)
{
	simple_lock_init(&_dybl_lock, 0);

	/*
	 * dynamic blacklist entries via boot-arg. Syntax is:
	 *  kasan.bl=kext1:func1:type1,kext2:func2:type2,...
	 */
	char buf[256] = {};
	char *bufp = buf;
	if (PE_parse_boot_arg_str("kasan.bl", bufp, sizeof(buf))) {
		char *kext;
		while ((kext = strsep(&bufp, ",")) != NULL) {
			access_t type = TYPE_NORMAL;
			char *func = strchr(kext, ':');
			if (func) {
				*func++ = 0;
			}
			char *typestr = strchr(func, ':');
			if (typestr) {
				*typestr++ = 0;
				type = map_type(typestr);
			}
			add_blacklist_entry(kext, func, type);
		}
	}
```

There is also useful tooling for LLDB to interact with KASAN when debugging. This is contained in 'tools/lldbmacros/kasan.py'. 

# UBSAN

Apple's Kernel implementation of [UBSan](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html) is also included within the ubsan.c source file. 

This is also controllable through sysctl's: 

```bash
boxname user$ sysctl -a|grep ubsan
kern.ubsan.logsize: 2048
kern.ubsan.logentries: 0
```
It is possible to dump the ubsan log file using 'sysctl kern.ubsan.log'.

# KSANCOV

Having code coverage feedback information exposed from the kernel allows for coverage guided based fuzzing.  On KASAN kernel builds, there is a driver interface exposed for KSANCOV called '/dev/ksancov'. The implementation for this is within san/ksan.c source file. 

There is also a test utility within san/tools/ksancov.c which can be used to obtain the information from the running kernel (provided it is built correctly!). 

The general process for a userspace program to obtain coverage data from the kernel is as follows:

```c
/*
 * ksancov userspace API
 *
 * Usage:
 * 1) open the ksancov device
 * 2) set the coverage mode (trace or edge counters)
 * 3) map the coverage buffer
 * 4) start the trace on a thread
 * 5) flip the enable bit
 */
```

Unfortunately, with the KASAN kernel binary from the KDK it is not built with KSANCOV support. Therefore, it would be necessary to build this kernel from source to use KSANCOV. 

```c
ifeq ($(KSANCOV),1)
# Enable SanitizerCoverage instrumentation in xnu
SAN = 1
KSANCOV_CFLAGS := -fsanitize-coverage=trace-pc-guard
CFLAGS_GEN += $(KSANCOV_CFLAGS) -DKSANCOV=1
endif

Symbols not present in kernel.kasan binary:

```c
___sanitizer_cov_trace_pc_guard
___sanitizer_cov_trace_pc_guard_init
___sancov.module_ctor_trace_pc_guard
```
