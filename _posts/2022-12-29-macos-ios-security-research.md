---
layout: post
title:  Vulnerability Research Digest - Issue 1 (macOS/iOS in 2022)
categories: [Apple,XNU]
---

In the past few years I created some twitter threads (e.g. [Windows Kernel Security](https://twitter.com/alexjplaskett/status/1553738346391822336) [Linux Kernel Security](https://twitter.com/alexjplaskett/status/1535189987846668288)) on a number of publications I found the most interesting within the vulnerability research space, this didn't really give me that much space to actually provide detail or allow this to be stored within a format which is easily accessible and I could refer back too. Therefore this years vulnerability research digest is going to be on my blog too.  

In this post I will cover some of the presentations I found most interesting from a macOS/iOS kernel security research perspective in 2022.  

# macOS/iOS Kernel Security

## Apple Neural Engine from POC 2022 ([@_simo36](https://twitter.com/_simo36))

https://github.com/0x36/weightBufs/blob/main/attacking_ane_poc2022.pdf

The Apple Neural Engine is the name for the energy efficient and high-throughput engine for Machine Learning inference on Apple Silicon. There has been previous security research into this by [Wish Wu](https://i.blackhat.com/asia-21/Friday-Handouts/as21-Wu-Apple-Neural_Engine.pdf) which describes the architecture and different components within both the software and hardware stack. _simo36's research sets the background on the frameworks and tooling which is used to interface with the engine and the IPC communication mechanisms used. It then describes the compiler used to build ML models and the pipeline used. The kernel interface is then documented with all its attack surface and shows the path right through from userland to kernel, then to firmware. 

_simo36 found a significant number of vulnerabilities within different components within the ANE architecture. He first describes multiple different classes of vulnerability for the issues he found (OOB write, integer overflow, improper index validation, signature check bypass etc). 

The talk then moves into exploitation and building an arbitrary r/w primitive by chaining together 4 different vulnerabilities. 

He also release the exploit for it here https://github.com/0x36/weightBufs

## Tales from the iOS/macOS Kernel Trenches from Zer0con 2022 ([@jaakerblom](https://twitter.com/jaakerblom))

https://github.com/potmdehex/slides/blob/main/Zer0Con_2022_Tales_from_the_iOS_macOS_Kernel_Trenches.pdf

This talk covers some recent vulnerabilities patched within macOS/iOS. Specifically multiple issues in IOMobileFramebuffer and one in the XNU kernel itself. 

The vulnerabilities discussed were (in IOMobileFramebuffer):
* CVE-2021-30883 - An integer overflow leading to a heap overflow reached from set_block selector
* CVE-2021-309XX - Multiple vulnerabilities also reached from the set_block selector and usable within Safari until 15.2 beta 3. 

The talk discusses about how IOMobileFramebuffer is mostly just a wrapper for passing data to the DCP hardware. Bugs actually in the DCP firmware and not the kernel code itself. He then talks about weaknesses within the DCP lack of mitigations and how DCP exploitation leads to traditional kernel r/w primitives. 

See Ian Beer's talk for a lot more info about this (as documented below). 

* The next vulnerability discussed is CVE-2021-30937 a core XNU bug. A set of racing bugs in setsockopt with improper locking which lead to a UAF.  

Project Zero describes the bug in this [issue tracker entry](https://bugs.chromium.org/p/project-zero/issues/detail?id=2224&q=CVE-2021-30937&can=1).   

The talk then focuses on the primitives gained from this bug (a byte copy primitive) and how they can be used for exploitation. Throughout the different versions of iOS different techniques can be used to spray. Therefore this section of the talk discusses Recent Generic Kernel R/W primitives:
* ipc_port
* pipe buffers
* uio
* IOSurfaceClient (the method used by Simo too). 

The talk then goes into discussing the different "capabilities" an exploit developer would require for exploitation, such as: 
* How to kfree and iOS 15. 
* How to know where controlled kernel memory is. 

The next section details exploitation of the following CVE's:
* CVE-2021-30883 - Using IOSurfaceClient technique (vuln described in https://saaramar.github.io/IOMFB_integer_overflow_poc/)
* CVE-2021-30937 - multicast_bytecopy (https://github.com/potmdehex/multicast_bytecopy)

The final section focuses on recent kernel mitigations on iOS 15.2-15.5 and how these can hinder an attacker. 
 
## Understanding Mach IPC from MOSEC 2022 ([@realBrightiup](https://twitter.com/realBrightiup/))

https://github.com/brightiup/Trekking/tree/main/Slides

This talk starts with a Case Study of Project Zero's issue 2107. A type confusion within the turnstiles leading to the kernel treating a `host_notify_entry` as an `ipc_port`. 

The second case study is from [Tielei](https://twitter.com/WangTielei) with another type confusion where it was possible to mislead the kernel into believing the bound destination port in a special reply port is of type `ipc_importance_task_t`. 

There are some key takeaways which can be draw from these issues. 

The talk then coverts the background knowledge of mach ports and IPC (and how rights work). The talk then goes into a deep dive into how the XNU kernel manages IPC, how the locks are critical and the details of different send / receive rights. 

I am not going to cover this one in detail, as Mach IPC is complex and the slides do a much better job of explaining it than I could. 

## AppleAVD from Hexacon 2022 ([@isciurus](https://twitter.com/isciurus/) [@NikitaTarakanov](https://twitter.com/NikitaTarakanov/)) 

https://github.com/isciurus/hexacon2022_AppleAVD/blob/main/hexacon2022_AppleAVD.pdf

This talk is about the Video decoding subsystem called AppleAVD. The talk starts off with an overview of the architecture and how the components fit together. The main focus of the talk is on the kernel component AppleAVD kext. The talk goes through analysis of this kext, the decoders implemented for parsing media within kernel space. The attack surface of the kext is analysed with its entry points and methods used for decoding / processing media. 

The talk then goes through previously found vulnerabilities within AppleAVD both in-the-wild and issues found by other researchers. The talk then talks about their process for analysis and bug hunting within the kext's (from both dynamic and static approaches). 

They then talk about the process of fuzzing the decoders. 

At the time of writing it seemed they didn't find any vulnerabilities within the decoder implementation themselves from fuzzing. However, they identified [CVE-2022-46694](https://support.apple.com/en-us/HT213530) a vulnerability within the outer kext logic which has now been patched by Apple. 

## Fugu15 - A deep dive into iOS 15 exploitation at OBTS ([@LinusHenze](https://twitter.com/LinusHenze/))

This talk covers the creation of a jailbreak for iOS 15 (including 15.2 and up). Creation of a jailbreak has become significantly more difficult where one kernel vulnerability was previously enough, now PAC and PPL bypasses are required as well. The talk describes the vulnerabilities used in Fugu15, how the kernel exploit works and PAC and PPL bypasses. It also covers new mitigations introduced in 15.2, what effect these have on jailbreaking and how these were bypassed in Fugu15. 

https://www.youtube.com/watch?v=rPTifU1lG7Q

https://github.com/pinauten/Fugu15

Notes on the four vulnerabilities used were as follows:

### fastPath (code signing bypass)
 * CMS signed by a certificate
 * CoreTrust ensures that CMS blob is valid and that hash matches code signature
 * Returns flags to AMFI to indicate cert type.
 * CT returns success as long as the CMS blob looks valid even if signed is not trusted (not a vuln). 
 * AMFI rejects if the CT validation fails otherwise it checks the CT flags 
 * If its app store signed then this will be set a valid. 
 * userspace amfid then has a chance to verify the signer is trusted.  

The vuln is that the flags are taken directly from the certificate (AppStore OID included in cert). No further checks on the signer are performed..  

This was a regression (iOS 13 and below not affected). 

  ** installHax - Wasn't fully patched (https://github.com/pinauten/Fugu15/tree/master/Tools/installHaxx). 

The code for this exploit is here: https://github.com/pinauten/Fugu15/tree/master/Exploits/fastPath

This gets arb code exec on the device. 

### oobPCI (arbitrary kernel r/w)

* Vuln in DriverKit (drivers in userspace). 
* Code signature bypass to grant the DriverKit entitlement. 
* Low level access to PCI (for reading and writing PCI devices). 
* Attacker controlled offset for a memory read/write (with no checks to determine if its within the mapping). 
* PCI Memory Address used for mapping is deterministic. 

Exploitation:
* Guess the offset to the physmap > Find the boot args region > Scan the initial page tables > Determine the offset of start address physmap and the current read address.  
* As the physmap is located at an L3 boundary, can calculate the lower 25 bits of the PCI region via offset to the Physmap.
* Then we can scan the RAM to find IOMemoryMap corresponding to the PCI memory (using the low 25 bits calculated previously) to determine the PCI memory start address and determine kernel slide from this and the vtable. 

However, on 15.2+ there are still mitigations which hinder full usage of the arb kernel r/w. Linus then goes on to talk about how he bypassed these as well with the following two vulnerabilities. 

The code for this exploit is here: https://github.com/pinauten/Fugu15/tree/master/Exploits/oobPCI

### badRecovery (CFI/PAC bypass)

A CFI/PAC bypass via thread fault handlers (CVE-2022-26765). 

As a reminder, Pointer Authentication Codes are a cryptographic signature for pointers which is used to prevent modification of the pointer/data. They are also used to implement control flow integrity (to prevent control flow hijacking). Thread fault handlers are a way to handle expected faults during data accesses. Kernel jumps to the handler when a data abort fault occurs.

These are typically used for things like copying data into kernel space (copyin), when an address would typically cause a panic if the thread fault handler was not set to handle this.   

Linus identified a function where it does a normal return and not an authenticated return (and you can't assume that the link register has the expected value and has not been modified).

I won't go into the details of the exploitation here, as the process of exploitation is well explained within the video recording. 

### tlbFail (PPL bypass)

PPL bypass via improper TLB flush

The final vulnerability within the chain and to bypass the last mitigation is a bypass of PPL. To recap PPL is the Page Protection layer which protects signed userspace code and some kernel data from being modified. It is a higher privilege level than the kernel. 

PPL also manages the page tables to prevent the kernel from mapping PPL-protected data and provides some methods for mapping and unmapping memory (but with checks to ensure your not trying to map PPL-protected memory).

Nested Page Tables
* Large DYLD shared cache mapped into every process
* Page tables are reused (shared) across multiple processes to save memory. 

Translate Lookaside Buffer
* Caches virtual to physical address translations
* TLB needs flushed when changing page table entries to prevent CPU using cached entries. 
* TLB entries have an address space ID (ASID) to limit flushing of the entries for only the page tables modified

He then covers the code for the logic of flushing the TLB. 

The question is what happens in the case of a PPL TLB flush? He then manages to modify the mappings in such a way that one process has access to Page X via the TLB (but with a refcount of zero) and to tell PPL that it now owns this page and forces reuse of it (as an L3 page table). Using this stale TLB entry it is possible to map page X by creating a new translation table entry and thus allowing mapping PPL protected memory.  

The talk finishes off by demo'ing the exploit. 

## The Journey To Hybrid Apple Driver Fuzzing from POC 2022 ([@Peterpan0927](https://twitter.com/Peterpan0927/))

https://github.com/star-sg/Presentations/blob/main/POC%202022/Zhenpeng%20Pan.pdf

The talk starts off by covering Apple Security Enhancements from a high level for the XNU kernel and the fact that these mitigations make exploiting UAF and type confusion much harder or unexploitable in certain scenarios. Therefore proposed that drivers will be a much more valuable target than before. 

He reviewed many POCs for drivers with the idea of building a fuzzer using code auditing and fuzzing together. The first tier of the fuzzer does lightweight collection of all reachable services and generates a second tier to do enhanced fuzzing based on code audit.   

He found multiple bugs this way, such as CVE-2021-30923/CVE-2022-22661/CVE-2022-32814 and over 15+ DOS bugs with others in the pipeline. 

Finally he covered some details of non public bugs found by the fuzzer and future plans together with an vulnerability which gives a 100% stable info leak of the kernel slide on macOS 13 together with a novel info leak attack surface. 

This talk was interesting for me, as back in 2017 (https://github.com/alexplaskett/Publications/blob/master/mwri-44con-biting-the-apple-that-feeds-you-2017-09-25.pdf) I developed a fuzzer using similar hybrid approaches which was also effective in finding multiple issues in the kernel. However, this talk highlights recent vuln's found and the code patterns which can be implemented in the fuzzer or reviewed for to find.   

## Abusing iPhone coprocessors for Privilege Escalation at OBTS ([@i41nbeer](https://twitter.com/i41nbeer/))

https://www.youtube.com/watch?v=H5oz1U03U1Q

We are starting to see much more interest into the separate co-processors, peripherals or firmware recently on mobile devices. This includes in-the-wild attacks. Ian's talk provided a technical deep-dive into how this in-the-wild exploit escaped the iPhone sandbox via a novel 0-day in the custom Apple co-processor. 

* Starts off the talk showing how a user is phished into installing a fake mobile operator application which is signed using an enterprise certificate. 
* Curious attack because it looked like the mobile network data was actually being disconnected via the operator. 
* Reversing the application showed that it was containing multiple exploits (including public ones from GitHub etc). sock_puppet, time_waste, lio_listio, AVE decoder
* However, contained two more 0-day exploits (when the sample was found).. 
  * CVE-2021-30883 (https://saaramar.github.io/IOMFB_integer_overflow_poc/)
* And the DCP 0-day the talk was about.. (CVE-2021-30883)

* Exploit had a bunch of printf statements in it which hinted how it worked, especially DCP (display co-processor) which handles low level display stuff on a separate piece of Apple silicon. 
* ARM64 Mach-o binary for the firmware 
* IOMobileFramebuffer functionality got moved out from the kernel into the DCP.
* A lot of the code literally got moved into the RTKit world. 
* At the time of the talk (don't know if this has changed now!) it had No PAC, ASLR, Predictable Heap Addresses,  

* Ian then goes on to describe how you communicate with the DCP. 
* Userland > Kernel > DCP (Deserialized and reserialized in the process). 
* DCP and DDR controller 
* IOMMU / SMMU / DART (he the describes how an IOMMU works) and how it can be used as a security boundary. 

* EL1 kernel code to DCP conversion.
* Exploit was building fake objects on the DCP to make kernel helper RPCs to map arbitrary memory such that from userspace they could read and write kernel memory. 

Then finally he talks about the actual vulnerability itself:
* Uniformity Compensation 
* Memory corruption described here https://googleprojectzero.blogspot.com/2022/06/curious-case-carrier-app.html
* Mitigations regression or engineering trade-off's moving onto low powered cores.  
* DCP/AVE/SEP/AOP/U1/AGX/ANE/BLE/WiFi (other co-processors / dedicated radio hardware etc) 

# Others presentations:

There are some other talks I didn't have time to cover in this issue but are also worth checking out. 

* Life and death of an iOS attacker by [Luca Todesco](https://twitter.com/qwertyoruiopz) - https://www.youtube.com/watch?v=8mQAYeozl5I
* A journey of hunting macOS kernel vulnerability by Peter NGUYỄN Vũ Hoàng - https://github.com/star-sg/Presentations/blob/main/Zer0Con%202022/A%20Journey%20Of%20Hunting%20macOS%20kernel.pptx
