---
layout: page
title: Research
permalink: /research/
---

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

