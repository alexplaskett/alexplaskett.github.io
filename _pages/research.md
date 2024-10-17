---
layout: page
title: Research
permalink: /research/
---

The following security research was performed whilst at NCC Group between 2021-now. 

## Listen-Up: Sonos Over-The-Air Remote Kernel Exploitation and Covert Wiretap

On the 8th of August 2024, Alex Plaskett and Robert Herrera presented this talk at BlackHat 2024 in Las Vegas, USA. 

[Slides](https://i.blackhat.com/BH-US-24/Presentations/US-24-Herrera-Listen-Up-Sonos-Over-The-Air-Exploitation-and-Covert-Wiretap-Thursday.pdf)

[Whitepaper](https://www.nccgroup.com/media/uzbp3ttw/bhus24_sonos_whitepaper.pdf)

The abstract of the talk was as follows:

Over the last year NCC Group found and exploited many different vulnerabilities within Sonos devices. This led to an entire break in the security of Sonos's secure boot process across a wide range of devices and remotely being able to compromise several devices over the air.

We leveraged these vulnerabilities to perform hidden recordings of the microphone to demonstrate how a remote attacker could be able to obtain covert audio capture from Sonos devices.

In this talk, we will start off with an introduction to Sonos devices, and describe the device architecture and security controls implemented (such as secure boot and disk encryption).

Then we will move into a deep dive on the Wi-Fi driver architecture and attack surface on the Sonos One. The talk will then describe a vulnerability we identified in the WPA2 Handshake which can allow a remote attacker to compromise the kernel over the air.

The talk will then move to the exploitation of this issue and discuss the novel challenges of developing a remote kernel exploit. To wrap up this section, we will then perform a demo of the attack where we will turn the device into a wiretap capturing all the audio within the physical proximity of the compromised device.

Finally, we will discuss vulnerabilities and exploitation techniques that allowed us to develop the world's first "jailbreak" of Sonos's flagship device - the Era-100 by breaking the secure boot chain. This affected 23 Sonos products and allowed the extraction of cryptographic material.

## Exploit Engineering – Attacking the Linux Kernel

On the 19th of May 2023, Alex Plaskett and Cedric Halbronn presentered this talk at Offensivecon 2023 in Berlin. 

[Slides](https://research.nccgroup.com/wp-content/uploads/2023/05/exploit-engineering-linux-kernel.pdf)

[Video](https://www.youtube.com/watch?v=9wgHENj_YNk)

The abstract from the talk was as follows:

Over the last year the Exploit Development Group (EDG) at NCC Group found and exploited three different 0-day Linux kernel local privilege escalation vulnerabilities (CVE-2022-0185, CVE-2022-0995, CVE-2022-32250) against fully patched OSs with all mitigations enabled. The most recent vulnerability was patched against versions of the kernel going back 6 years affecting most stable Linux distributions.

Unlike developing proof of concepts, our exploits need to be ultra-reliable and support many different OS variations and kernel versions so they can be used by our security assessment consultants or Red Teams. This calls for a much more rigorous engineering process to be followed.

In this talk, we start with an overview of our bug hunting processes and approach to rapidly find high impact vulnerabilities within the Linux kernel. The talk will then describe key vulnerability details, discuss the challenges of reliable exploitation across multiple targets and describe the exploitation techniques used (and what is appropriate in 2023). We discuss rigorous exploit engineering approaches – including tooling which we have developed for heap analysis [libslub](https://github.com/nccgroup/libslub) and automation for mining, creation, deployment and scaling across many different environments (TargetMob). Finally, we will conclude with our thoughts on areas where more strategic hardening and attack surface reduction can be introduced to hinder against advanced attackers using 0-days in the Linux kernel. We will leave you with a release of our tooling for heap analysis [libslub](https://github.com/nccgroup/libslub) and the knowledge to go out there and find, analyse and exploit your own Linux kernel vulnerabilities!

This research was performed by Alex Plaskett, Cedric Halbronn, Aaron Adams and presented at [Offensivecon 23](https://www.offensivecon.org/speakers/2023/alex-plaskett-and-cedric-halbronn.html).

## Your Not so “Home” Office – Soho Hacking at Pwn2Own

On the 20th April 2023, Alex Plaskett and McCaulay Hudson presented this talk at HITB AMS. The talk showcased NCC EDG in Pwn2Own 2022 Toronto targeting all consumer routers (Netgear, TP-Link and Synology) from both a LAN and WAN perspective.  The talk also described how we compromised a small business device (Ubiquiti) via the WAN and used that to pivot to attack a device on the LAN (a printer). In total we created 7 different exploit chains and found many more vulnerabilities within the process.

[Slides](https://research.nccgroup.com/wp-content/uploads/2023/04/D1T1-Your-Not-So-Home-Office-Soho-Hacking-at-Pwn2Own-McCaulay-Hudson-Alex-Plaskett.pdf)

[Video](https://www.youtube.com/watch?v=vvUrSHLsNxI)

This research was performed by Alex Plaskett, Cedric Halbronn, Aaron Adams, McCaulay Hudson and presented at [HITB AMS 2023](https://conference.hitb.org/hitbsecconf2023ams/).

## Toner Deaf – Printing your next persistence

In November 2021, NCC Group won at the Pwn2Own hacking contest against a Lexmark printer. This talk is about the journey from purchase of the printer, having zero knowledge of its internals, remotely compromising it using a vulnerability which affected 235 models, developing a persistence mechanism and more.

This talk is particularly relevant due to printers having access to a wide range of documents within an organisation, the printers often being connected to internal/sensitive parts of a network, their lack of detection/monitoring capability and often poor firmware update management processes.

[Slides](https://research.nccgroup.com/wp-content/uploads/2022/10/toner-deaf-hexacon-2022-release.pdf)

[Video](https://www.youtube.com/watch?v=TUHcZptN6Jk) 

This research was performed by Alex Plaskett, Cedric Halbronn, Aaron Adams, Catalin Visinescu and presented at [Hexacon 2022](https://www.hexacon.fr/conference/speakers/#toner_deaf).

##  Pwn2Own 2021 - Remotely Exploiting 3 Embedded Devices

This research demonstrates the process taken and vulnerabilities used for [Pwn2Own Austin 2021]([https://www.thezdi.com/blog/2017/11/2/the-results-mobile-pwn2own-2017-day-two](https://www.zerodayinitiative.com/blog/2021/11/1/pwn2ownaustin)) which were used to compromise a Western Digital PR4100 NAS, a Lexmark MC3224i and outside of the competition a Netgear R6700v3.   

[Remotely Exploiting 3 Embedded Devices Slides](https://research.nccgroup.com/wp-content/uploads/2022/07/pwn2own-3-bugs-technical-external.pdf)

[How to win $$$ at a hacking contest? Slides](https://research.nccgroup.com/wp-content/uploads/2022/07/pwn2own-how-to-win-external.pdf)

[Western Digital Blog](https://research.nccgroup.com/2022/03/24/remote-code-execution-on-western-digital-pr4100-nas-cve-2022-23121/)

[Lexmark MC3224i Blog](https://research.nccgroup.com/2022/02/18/analyzing-a-pjl-directory-traversal-vulnerability-exploiting-the-lexmark-mc3224i-printer-part-2/)

[Netgear R6700v3 Blog](https://research.nccgroup.com/2022/02/28/brokenprint-a-netgear-stack-overflow/)

This research was performed by Alex Plaskett, Cedric Halbronn, Aaron Adams, Catalin Visinescu and presented at NCC Con Europe 2021. 

## Pwning the Windows 10 Kernel with NTFS and WNF

A local privilege escalation vulnerability (CVE-2021-31956) 0-day was identified as being exploited in the wild by Kaspersky. At the time it affected a broad range of Windows versions (right up to the latest and greatest of Windows 10). With no access to the exploit or details of how it worked other than a vulnerability summary the following plan was enacted:

1. Understand how exploitable the issue was in the presence of features such as the Windows 10 Kernel Heap-Backed Pool (Segment Heap).
2. Determine how the Windows Notification Framework (WNF) could be used to enable novel exploit primitives.
3. Understand the challenges an attacker faces with modern kernel pool exploitation and what factors are in play to reduce reliability and hinder exploitation.
4. Gain insight from this exploit which could be used to enable detection and response by defenders.

The talk covered the above key areas and provides a detailed walk through, moving from introducing the subject, all the way up to the knowledge which is needed for both offense and defence on modern Windows versions.

[Slides](https://research.nccgroup.com/wp-content/uploads/2021/11/poc_2021_pwning_the_windows10_kernel_final_slides.pdf)

The following security research was performed whilst at MWR InfoSecurity (now F-Secure Consulting) between 2011-2018.

## Big Game Fuzzing - Pwn2Own Apple Safari

This research describes the vulnerabilities used for [Pwn2Own Desktop 2018](https://www.thezdi.com/blog/2018/3/15/pwn2own-2018-day-two-schedule) to compromise Apple macOS Safari. It describes the tools developed and the process taken in order to identify these vulnerabilities. The slides and whitepaper also describe the exploit development process and techniques used for exploitation of the vulnerabilities. The vulnerabilities described within these documents are a Wasm vulnerability (CVE-2018-4121), an SVG vulnerability (CVE-2018-4199) and a sandbox escape within the Dock component (CVE-2018-4196). 

[Slides](https://github.com/alexplaskett/Publications/blob/master/mwri-t2-big-game-fuzzing-pwn2own-safari-final.pdf)

[Whitepaper](https://github.com/alexplaskett/Publications/blob/master/apple-safari-pwn2own-vuln-write-up-2018-10-29-final.pdf)

This research was performed by Fabian Beterke, Georgi Geshev and Alex Plaskett and presented at T2 2018. 

## The Mate Escape - Huawei Pwn2Own 

This research demonstrates the process taken and vulnerabilities used for [Pwn2Own Mobile 2018](https://www.thezdi.com/blog/2017/11/2/the-results-mobile-pwn2own-2017-day-two) which were used to compromise a Android Huawei Mate 9 Pro device. The vulnerabilities used within this chain were logic type bugs and no memory corruption issues were used. Whilst memory corruption protections and mitigations are offering additional protection to the platform, logic bugs are often neglected and can be used to equally damaging effect. 

[Slides](https://github.com/alexplaskett/Publications/blob/master/huawei-mate9pro-pwn2own-write-up-final-2018-04-26.pdf)

[Whitepaper](https://github.com/alexplaskett/Publications/blob/master/huawei-mate9pro-pwn2own-write-up-final-2018-04-26.pdf)

[Video](https://www.youtube.com/watch?v=-eAR6qduVWY)

This research was performed by Alex Plaskett and James Louerio and presented at Snoopcon 2018, Hacktivity 2018. 

## Apple Safari - Wasm Section Exploit 

This whitepaper describes the process taken when investigating a potential vulnerability for Pwn2Own. Web Assembly was a relatively new feature added the browser and therefore was expected to not have undergone as much security assurance at other areas. Unfortunately whilst performing exploit development of the issue, the issue was fixed by Apple (and therefore would not qualify for Pwn2Own). The issue was addressed publicly with macOS 10.13.4 and was found independently by Natalie Silvanovich of Google Project Zero.

[Whitepaper](https://github.com/alexplaskett/Publications/blob/master/apple-safari-wasm-section-vuln-write-up-2018-04-16.pdf) 

This research was performed by Fabian Beterke, Alex Plaskett and Georgi Geshev in 2018 and presented at T2.fi. 

## Biting the Apple That Feeds You - macOS Kernel Fuzzing

This research demonstrated techniques for macOS kernel fuzzing in order to find security issues with macOS. Previously, only a small amount of research had been published about automating finding vulnerabilities within the macOS kernel. The slides describe the tooling which was developed and the issues which were found (and addressed by Apple) as part of this research. The slides demonstrate that different fuzzer approaches can lead to different vulnerabilities being found. macOS IPC subsystem was also discussed and tooling produced to target these features. 

[Slides](https://github.com/alexplaskett/Publications/blob/master/mwri-44con-biting-the-apple-that-feeds-you-2017-09-25.pdf)

[Video](https://www.youtube.com/watch?v=TA_sQk2oiqU)

This research was performed by Alex Plaskett and James Louerio and presented at Warcon 2017, 44CON 2017, Deepsec 2017.

## QNX - 99 Problems But A Microkernel Ain’t One!

This research investigated the security of the QNX operating system and outlined methods for finding vulnerabilities in this area. There are a large number of devices which run QNX under the hood. These are often Cars, Turbines and Safety Critical Systems, therefore the security of these devices in paramount. This research focused on Blackberry 10’s version of QNX, however, this research is applicable to all QNX based devices. The slides and whitepaper provide an overview of the operating system, our methods for identifying vulnerabilities and any issues identified. The research also described how the subsystems on QNX communicate and methods an attacker may used to perform privilege escalation across the trust boundaries.

[Slides](https://github.com/alexplaskett/Publications/blob/master/mwri-qnx-troopers-99-problems-but-a-microkernel-aint-one_2016-03-19.pdf)

[Whitepaper](https://github.com/alexplaskett/Publications/blob/master/mwri-qnx-security-whitepaper-2016-03-14.pdf)

[Video](https://www.youtube.com/watch?v=ump5KV2tD6U)

This research was performed by Alex Plaskett and Georgi Geshev and presented in 2016 at Conﬁdence 2016, Troopers 16, BSides NYC 2016. 

## Windows Phone 8 - Navigating A Sea Of Pwn

This research investigated the security of Windows Phone 8 applications and described methods which could be used to test them. Whilst Windows Phone 8 is now a deprecated platform, at the time it was Microsoft's latest mobile operating system. This research shows approaches which can be taken when assessing a Windows Phone 8 application and potential security issues which can arise.

[Slides](https://github.com/alexplaskett/Publications/blob/master/mwri_wp8_appsec-slides-syscan_2014-03-30.pdf)

[Whitepaper](https://github.com/alexplaskett/Publications/blob/master/mwri_wp8_appsec-whitepaper-syscan_2014-03-30.pdf)

[Video](https://www.youtube.com/watch?v=sUBnhCgSVew)

The research was performed by Nick Walker and Alex Plaskett and presented at Syscan and Qualcomm Security Summit in 2014. 

## Windows Phone 7 - Owned Every Mobile

This talk presented the research performed into Windows Phone 7 and demonstrated one of the first browser (Internet Explorer) exploits against the platform. It demonstrated weaknesses the OEMs had also introduced into the platform and demonstrated methods to bypass sandbox restrictions. 

[Slides](https://github.com/alexplaskett/Publications/blob/master/mwri_wp7-bluehat-technical_2011-11-08.pdf)

[Video](https://www.youtube.com/watch?v=pOVVFM_x980)

The research was presented at 44CON, T2.ﬁ, DeepSec and Microsoft BlueHat in 2011. 

## Windows Phone 7 - Microsoft BlueHat v11 Executive Briefings

This talk presented a higher level view of the security research performed against Windows Phone 7 to a number of Microsoft Execs during BlueHat v11. 

[Slides](https://github.com/alexplaskett/Publications/blob/master/mwri_wp7-bluehat-exec_2011-11-08.pdf)

This research was presented at Microsoft's BlueHat Executive Briefings in 2011. 

