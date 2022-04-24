---
layout: post
title:  Demystifying Security Research - Part 1
categories: [Research,Mindset,Approach] 
---

There are a number of key questions which are always asked by people wanting to get into security research, find out more about how others go about it or just generally improve their processes. In this post I want to highlight some of things which work for me and some guidance which may help for others. This is a rare less technical post by me as it feels like a lot of the time people see the end results of the research but the process of getting there and the challenges faced is less obvious. 

Some of the commonly asked questions asked are:

* How to get started within security research? 
* What topics do you think are worth researching?
* How do you approach a new topic initially?  

There is no one size fits all answer for this and a lot of this post is just what works for me. However, there are some key points to consider and tips to give. This is mainly written from the perspective of a non-academic offensive security researcher, who has done security research both for a free time hobby and as a career. [XPN](https://twitter.com/_xpn_/status/1515987957894848513) started a twitter thread on this recently and this just goes to show how varying sources of motivation inspire others. [Yarden Shafir](https://medium.com/@yardenshafir2/security-research-and-the-creative-process-552fd91f52a7) also detailed the creativty in security research.  

From a career perspective, I also think this is one of the paths which is less clear to get into and what exactly "security research" is. There's also a wide variety of what you would call "security research". Some do this entirely in their free time, others dedicate certain time components to it and others are fortunate to have it as their full time career. There's also the fact that this spans from something like bug hunting to all the way to academic publications and professorships. Regardless of what you consider security research to be, there are some key takeaways when working on more novel security areas. 

I will aim to cover the first two points in this article, part 2 will cover the approach in more detail. I also will give a concrete demonstration of a real research project and the approach taken.  

# Topic Selection

Probably the most important thing when embarking on security research is what topic should this be in? 

Here are some key areas which may help guide this train of thought.  

## Enjoyment 

*Look into something which you care about* 

This is the most important thing! You are not going to do your best work if you are not interested in the subject or you are led down a path by someone else which is not right for you. There are lots of reasons why people find inspiration or end up choosing the area they do and this is very individual. However, here are some examples: 

* See a technology on a pentest engagement which there has not been much published on or any tools available for? 
* Have you worked in another industry previously and have in depth knowledge in something unique or know people who do? 
* Listen to others who either encounter a problem in a certain area and need help. (This one's the more controversial one, as others can often have different incentives, however, if you can provide value and also find it enjoyable in the process, then this is a win win for both parties).  
* Proving a statement or assumption wrong. This is the typical "hacker" / adversarial mindset but massively helps with security research as it is basically providing assumptions or threat models wrong or inadequate. 
* Investigation of in the wild tools, techniques and procedures used by threat actors. 
* Spot an area for improvements either from a defensive or offensive perspective or a common problem an organisation has.   
* See a talk which inspires you to dig more into a subject. 

It is worth mentioning that a lot of more experienced researchers have way more ideas than they have time available to spend on these things. Whilst asking for advice is advantageous and can help focus initial thoughts and provide some background into what has been done before, the topic really needs to be driven by your own personal interests and curiosity. 

## What's hot and what is interesting to you

There will always be trendy topics within security research. In the past, mobile was a big thing, then cloud was flavour of the week, now Web3 is all over the twitterverse and conferences are starting to dedicate categories to it. 

The way I see it is that ultimately the technologies are still built on the fundamentals of computing. To have a non-surface level understanding of these technologies, the basics like programming, networking and operating systems are really important. Once you get started in one area, transferring those skills to the flavour of the week/month technology is very achievable. Sure, it may take some time, but in my opinion it is much harder to go the opposite way. 

Those who are more experienced security researchers can pick up a topic and make an impact very quickly based on all the prior knowledge. As a concrete example, reviewing web3 smart contract issues is massively aided by just generally understanding application security and common vulnerability classes. A lot of this is just being able to move up and down the abstraction layers.  

## Business Needs

Whilst a lot of security people get into security research mainly because of their interest and the fact that it is basically a hobby, a lot are very fortunate to have companies support the research or need it for product and service development. 

Obviously the more your research topic aligns with commercial needs, the more likely support and backing is available for you. This is a really large topic and really depends what the business purpose is. However, some things to think about could be:  
* How does the research contribute to services / product development / marketing? 
* Is the research going to enhance skills / knowledge etc?
 
In general though, the security research you do often takes a while to translate into tangible things and it's often difficult to directly correlate the two. For example, you may present a research topic and win business from it.. that is measurable and the impact obvious. However, you may also release something, feel like it's gone unnoticed, then many years down the line find out it led to the recruitment of someone or your whitepaper is the de-facto reference material for a certain subject. That's why I think it's important to take a longer term view of these things and not dwell too much on the immediate short term impact.

From a purely getting research done perspective, unless your job is primarily to manage and organise and shape the direction of this, then overthinking this can lead to procrastination and hinder productivity but it does help to have this in the back of your mind. 

# Brainstorming and Collaboration 

At the start of every project it makes sense to sit down and brainstorm about possible approaches and what has been done previously. This ties into the background research section I have covered below. However, here are some key points: 

* Discuss what you are thinking of doing with other people. They may highlight areas you may not have considered or may not be aware of. An example of this may be doing a whiteboard session. This also really helps when you face a setback or deadend. Some of the best exploit ideas have come from discussions with others having previously hit a wall. 

* In the past, a large amount of security was discussed on IRC. Times have changed and technologies have moved on. These days the most visible public discussions occur on twitter, discord and slacks. These mediums are good for building up initial dialog between researchers but networking at conferences and other face to face events is very advantageous when aiming to collaborate on a project. 

* Share your work and get feedback (CFP reviews etc). 

It's good to share your work early. If you are writing a CFP for a conference, get feedback from both people who know the area well and also those who know nothing about the area. Writing a good CFP entry itself is challenging and there are things which affect the chances of acceptance. 

# Motivation and Mindset 

Anyone who does vulnerability research knows it can be a rollercoaster of emotions. The research output you see at conferences often only shows half the picture and can gloss over all the failure in reaching that goal.  

The many weeks of not finding anything, thinking you found something, finding out something is not exploitable, finding out the bug you found just got patched etc.. The very temporary buzz you get from getting code exec. Then back again and repeat all over =) 

[Mark Dowd](https://twitter.com/mdowd/) OffensiveCon [presentation](https://www.slideshare.net/MarkDowd13/rules-to-hack-by-offensivecon-2022-keynote-251318003?qid=c6ba0071-d766-4694-9fc1-ca2dc1c1ea03&v=&b=&from_search=1) has covered a lot of this recently from a vulnerability researcher perspective and the slides are a very recommended read so its not worth duplicating here. In a future post I will talk about some of the failures in getting to the goal for a concrete topic. 

# Focus

One thing I need to cover for security research is focus and distractions. Whilst you can achieve reasonable results dipping in and out on a long term research project, personally I find too much context switching can massively impact my performance. With that said, sometimes it does make sense to switch to another project temporarily to achieve a sense of accomplishment after hitting a wall for so long. The knowledge built up from the initial project will be there and especially in cases such as actively developed codebases may lead to new vulnerabilities being identified which were not there previously. 

As a practical example, it is also very challenging to go from being deep within the Linux kernel and then switch to an entirely different platform temporarily. I will cover more about this in the note taking section below. 

There is also this concept of deep work vs shallow work. Deep work is the ability to focus without distraction on a cognitively demanding task. With so much of the modern world fighting for your attention it's very easy to perform shallow work. Whilst things like twitter, slack etc are super advantageous keeping up to date with what's going on. Notifications and random miscellaneous tasks can destroy this deep focus work. For this you need to build time management strategies which work for you. 

# Research Goals and Skills Growth

Having a good set of tangible practical goals for what you want to achieve is beneficial. However, I think you need to set these goals at a level which is appropriate and your existing knowledge level and experience. The goals need to be something which challenges you and are by no means guaranteed to be successful.

For someone just starting out in their research path then examples could be: 

1. Publish an article on a blog. 
2. Release a tool to do X. 
3. Find an 0day in software Y. 

Some examples of harder research goals could be:

1. Present your research at the top technical conf's in the industry.  
2. Write a book on the subject
3. Win Pwn2own in the X category. 
4. Aim to win a high prize in a top bug bounty (e.g. a perceived hard target).  
5. Develop a mitigation for X class of vulnerability 

Ultimately if you set your goals correctly and achieve them, then on the way there you will experience skills growth. 

# Impostor Syndrome

It is worth touching on this because I think everyone within the security industry feels this at some period of time, some more than others. 

When you start off researching and know very little about a subject, then there's obviously going to be people out there who know way more than you. Even after a long time researching a subject, there can be people who are hyper-specialised in that area who still know far more. 

Ultimately, there's always going to be someone who's more technical or likely knows more about you than a subject. (Although admittedly, in certain areas of really niche security research you may be one of the handful of people in the world =)). 

Therefore, I think it's important to look back and understand all the things you have achieved. Your research project may not go as well as you had hoped but there's always more to do and other areas to explore. Getting some short term tangible output from it can really help with the longer term motivation and feelings of accomplishment. 

Likewise, seeing other respected researchers within the industry ideally should be more motivational and help with the mindset of wanting to be "at that level" rather than being demotivational. I guess it's easy to say this but perhaps hard to realise this in practice. 

We stand on the shoulders of giants as they say! 

# Research Notes

Research note record keeping is a big thing, having tons of files called a.txt, b.txt, lolz.c, uaf.c is not really scalable after a while :) Since I started writing notes in markdown and using version control for tracking and development, then being able to come back, review and update at a later stage is super useful. As research is an iterative approach and that you may often reach a dead end, being able to go back and pick up different areas is super important. 

I find maintaining a list of previous research on a topic useful and try to do this for each area I look at. This is normally composed during the background research performed (section below) and updated once I become aware of new papers.  

Some good public examples of these which accept PRs are:
* [Linux kernel examples](https://github.com/xairy/linux-kernel-exploitation)
* [Exploit mitigations](https://github.com/nccgroup/exploit_mitigations)

# Background Research 

The first thing I do when embarking on a new research project is review all the material which is already out there on that subject. As an example, recently I had been working on Windows Kernel exploitation, so going through recent conference presentations, whitepapers and blog posts to understand whats already out there was super important. As I had more reccently been looking at macOS related stuff, my windows knowledge was a little rusty and its pretty hard to carry around multiple platforms security aspects in your head at all times =)    

Some of the main sources of information I use are:
1. Conference output 
2. Blog posts
3. Academic papers 
4. Bug tracking systems 
5. Source code history 

# Conclusion

In part 1 of this blog, I hope I raised some useful points and have helped demystify some of the thought processes which go on prior to and behind the research. Expect a more concrete example from a past research project in the near future.    

# References

[1] Mark Dowd - [OffensiveCon Key Note](https://www.slideshare.net/MarkDowd13/rules-to-hack-by-offensivecon-2022-keynote-251318003?qid=c6ba0071-d766-4694-9fc1-ca2dc1c1ea03&v=&b=&from_search=1) 

[2] Yarden Shafir - [Security Research and the creative process](https://medium.com/@yardenshafir2/security-research-and-the-creative-process-552fd91f52a7)

[3] XPN - [ramdom twitter thread](https://twitter.com/_xpn_/status/1515987957894848513)