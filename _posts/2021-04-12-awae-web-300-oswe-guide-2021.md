---
layout: post
title:  "Just another AWAE / WEB-300 / OSWE guide in 2021"
date:   2021-04-12
categories: WEB
thumbnail: /img/oswe.jpg
tags: web-300 oswe awae oswe-prep certifications
---


A few days ago I earned my OSWE certification and naturally, this calls for a write-up that many asked me to do! Without reiterating the same things and suggestions written better in some of the guides I read before my exam, I will link those in this post and only add some pointers that I think will be helpful. It never hurts with one extra write-up as you get another angle on the same thing and you may resonate better with the person's thinking! 

Before diving into my guide for this course, here are a few lines about my experience and the journey. Feel free to skip this paragraph if you are only interested in the guide! After I got done with my [OSCP journey](https://medium.com/@rayhan0x01/my-fast-paced-freemium-oscp-journey-26c5ccb2a8a3), I found myself creating web-based CTF challenges for various community CTF events. To write custom web challenges, I had to read vulnerable codes to understand why certain vulnerabilities occur to implement them on my challenges. I think this in particular helped me prepare for the OSWE course without even knowing! Fast forward a few months, I saw the $999 deal for 30 days lab of OSWE course. I was very fast-paced with my OSCP course and barely took everything the course had to offer in 30 days, I wanted to do the same again for OSWE so I jumped the wagon and got myself enrolled! My lab time started on 28th February and lasted till 30th March. Now You cannot purchase 30 days offer anymore, the minimum is 60 days and I think that's plenty of time. I was able to sift through my course content in just under 16 days all thanks to my previous experience working with OWASP top 10 vulnerabilities, automation, and scripting with Python. I have been informally coding in Python, PHP, JavaScript, and many other languages since I was 15 (now 20!) that has surely given me the edge to be able to quickly adapt to white box web application assessment methodology. Still, my weakest section on the course has to be not much exposure to MVC (model-view-controller) applications. But with little practice and patients, I was able to overcome the difficulties! Took my exam 5 days after the lab ending period, finished the exam in 47 hours, got my result in 24 hours that I passed. Also, it brings me great joy to share that I may very well be the first person or at least one of the few to earn OSWE in my country as I have not found anyone else who achieved this certification from my country. 

&nbsp;

Now coming to the main part of this post, I'll try keeping it as simple as possible with different sections below.


## # What is this course and what skills will I gain from this course?

I feel like Offensive security answered it best on their online badge issued in acclaim/credly. Find more about the course [here](https://www.offensive-security.com/awae-oswe/)

![oswe-credly](/img/oswe-credly.png#center)

&nbsp;

## # What should I need to know as pre-preparation before the course?

* Know how to script and automate at least one programming language (preferably Python). Specifically, interact with web applications such as submitting forms or brute-forcing endpoints, etc. Also, know how to multi-thread.

* Although not mentioned as a pre-requisite, you should at least be able to read code in other languages such as PHP, JavaScript, C#, and Java. I suggest learn the basics of these mentioned languages to a point where you can at least google and scrape together small scripts if needed.

* Know how MVC (model-view-controller) frameworks work. Maybe spend some time learning how Laravel in PHP or Django in Python works. Build a small web app as a practice if needed. If you are confident, you may skip this for now and learn about it during the course.

* Get familiar with OWASP top 10 vulnerabilities. You should be familiar with SQLi, XSS, LFI, RCE, SSTI, XXE. [PortSwigger WebSecurityAcademy](https://portswigger.net/web-security) is a great place to practice these vulnerabilities. I will link to two more great guides/write-up at the end that will contain links to real world applications for practice. If you want hardcore practice, here's an unlikely suggestion, solve active web challenges in HackTheBox! 

* Know regular expressions. You should be able to match and extract any particular data from a web page using regex.

* Get familiar with BurpSuite. You are only allowed to use the community edition so you should stick to the community edition.

These should be enough as pre-preparations for the course. **The course materials are enough to pass the OSWE exam.**
This has been asked so many times so I wanted to make it clear that yes it's enough.

&nbsp;

## # What should I do during my lab time?

* Unlike OSCP course, the course book and the videos goes hand-to-hand. You should not skip either one or you will miss important details.

* Do all the exercises. **Do the extra miles!** All the exercises that involves automation, do them! Try to make your scripts modular so you can re-use them in future if needed. I.E, if there is a blind SQL injection, try to make individual functions and SQLi query template strings so you can re-use the code with little modification if you find same vulnerability in a different application. If stuck, take help from the forum, [Infosec Prep Discord](https://discord.gg/mEtEFhp), [Offensive Security Discord Server](https://help.offensive-security.com/hc/en-us/articles/360049069012-Offensive-Security-Community-Chat-User-Guide).

* Create a checklist of all the vulnerabilities that you were taught during the course. Have notes on each vulnerabilities and how to find them. This will greatly help during the exam as you'll be able to work by process of elimination. As I said earlier, course materials are enough to pass the exam. Take note of logical vulnerabilities and the places where having them will allow you to exploit the application.

* Solve the extra lab machines. You'll be given multiple extra lab machines with no guide or instructions on how to solve them. There may be multiple ways you can solve a machine, if you have time then try solving in more than one way.

&nbsp;

## # What should I do during my exam?

* This is coming from my personal experience, the remote debugging through RDP won't be a comforting experience in the exam. Depending on your Internet speed, there maybe severe delay or lag in the output. My suggestion is to use &nbsp;`sshfs`&nbsp; to mount target application source code locally and then use VSCODE to open the mounted folder.

* Go through your checklist of vulnerabilities, don't fall into rabbit holes! Even if something you see may look vulnerable, if you can't reach the vulnerable portion of code then you cannot exploit it. Try having a scenario in mind for overall exploitation as soon as you confirm a vulnerability.

* If you find a vulnerability but don't know what to do with it then think about what are additional things you can gain from this vulnerability. Use the debugging machines to your advantage, look into the system, insert custom codes to save logs of something if necessary.

* Take breaks! 48 hours is a long time and you need to keep your head straight throughout this exam. When you take breaks to catch some air, you'll have new ideas to try. Sleep a little if possible.

* Do not give in till the end! You may think after a period of time that it's a loosing battle and you should give up but don't! You may think you looked at every place but you didn't, these applications are purposefully vulnerable so the vulnerability must exist! I personally had no progress in first 12 hours into the exam. But after 47 hours, I had full 100 points!

&nbsp;

I tried giving some useful pointers for the overall course and avoided referencing specific resources to practice as there have been many good write-ups on those. Here are two additional write-ups that will contain real world applications that you may practice or look at if needed:

* [https://z-r0crypt.github.io/blog/2020/01/22/oswe/awae-preparation/](https://z-r0crypt.github.io/blog/2020/01/22/oswe/awae-preparation/)

* [https://hub.schellman.com/blog/oswe-review-and-exam-preparation-guide](https://hub.schellman.com/blog/oswe-review-and-exam-preparation-guide)


So that was it for this one! If you have questions for me, feel free to reach out via Twitter or LinkedIn. 