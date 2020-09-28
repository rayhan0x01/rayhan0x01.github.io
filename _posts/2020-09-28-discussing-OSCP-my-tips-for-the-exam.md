---
layout: post
title:  "Discussing common OSCP issues and my tips for the exam! "
date:   2020-09-28
categories: OSCP
thumbnail: /img/oscp.jpg
tags: ctf,defcon,redteamvillage
---
Hey there! This post is for the folks who want to take on the OSCP exam. Some of the experiences I am sharing here might help you answer some of the questions you might have! If you want to read my OSCP journey, please [have a read at this post](https://medium.com/@rayhan0x01/my-fast-paced-freemium-oscp-journey-26c5ccb2a8a3?sk=87ba9d1c8c27cb0d5cae9db3305169ba)! Here I'll be discussing some of the common issues you might face during the exam, share some of my resources, and tips for someone just starting this journey!



## # Internet speed!

I know this is a concern for many people! I started the course with an internet speed of 5mbps and later I had to upgrade this to 30mbps to get a comforting experience. If your Internet connection speed drops lower than 3mbps you would be able to count the pixels getting loaded of a Windows remote desktop interface so yeah minimum 5mbps is important as specified in the Proctoring-Faq page.

## # WebCam!

I bought a nice Xiaomi 1080p Webcam to appear in the exam and realized on the exam day that it doesn't have autofocus! You have to show a Government Issued ID card in the webcam for verification and if your webcam doesn't have autofocus the text won't appear clearly!
I quickly downloaded the DroidCam app from PlayStore and had it installed on my computer as well. I asked the proctor if I can use my Android as a webcam for verification as the desktop webcam was not focusing the texts on my ID card and they agreed! Later I switched back to my desktop webcam for the rest of the exam. So I highly recommend having a WebCam that has an autofocus feature!

## # System Crash!

I was almost 100% sure I won't encounter a system crash during my exam like many others faced before me. My workstation was in a very good state but still, this had to happen during the second hour of my exam! I took a break and came back to see my computer is completely hanged! Partly the reason I guess was because of the DroidCam client I had running and when I left the room and disconnected my Phone that resulted in something bad. I did a force reboot, went to my exam portal, notified the proctors that my system crashed. I was told not to worry and continue with my exam. I think I lost about 15 minutes or more while this happened and was out of the reach of the proctors. The point I want to make is that don't get scared if something like this happens. You won't get disqualified for an issue like that as they can see the VPN traffic and at the time of your crash your activities will stop so they know you aren't in the network. Make sure to have a good snapshot of your VM as well before the exam day!

## # Electricity!

Yeah, we folks from Asia have to worry about that as well! Even though I didn't face any electrical outage during my exam I thought about it a lot and had a UPS as a backup so that I can inform my proctors and take a snapshot of my VM when it happens. Keep a backup internet supply like an internet package on your mobile so that you can come back online asap and notify them! If it takes too long for the electricity to come back your proctors may decide to reschedule the exam!

## # Linux as host machine!

I gave my exam on the latest Ubuntu LTS release. You would have to share the screen of your host machine and not the virtual machine. There's an issue of black screen when sharing the full screen on the latest Ubuntu via Google Chrome plugins. If you happen to be using Ubuntu on Wayland, switch to Ubuntu on Xorg to fix the issue. 
If you are using any other XYZ Linux distribution, you can request a test proctoring session where you can check your webcam and screen sharing just like the exam. The instruction is given at the proctoring-faq page.

## Sharing My Resources 

* You will find short write-ups on some of the common Windows exploits that are usable via reverse-shell/commandline without Gui access [Here](https://github.com/rayhan0x01/reverse-shell-able-exploit-pocs).
* My cheat-sheets on Linux and Windows commands and Windows Privesc can be found [Here](https://github.com/rayhan0x01/my-cmd-stash).

## For someone willing to take on OSCP
It has now become a tradition to pass on tips or learning resources from someone who passed the exam! Because all that we have learned are from the community and it finally feels good to share our experiences and suggestions hoping it may benefit someone next on the line!

* If you are just starting and want to take on OSCP I would suggest the following:
Get some Python basics. Even if you are good at some other language. Most exploit POC scripts you will be dealing with will be in Python. Also, it is important to be comfortable with both the Linux and Windows command-line interface, have basic knowledge over networking and DNS.
* For Buffer overflow basics, go through [TheCyberMentor's playlist](https://www.youtube.com/playlist?list=PLLKT__MCUeix3O0DPbmuaRuR_4Hxo4m3G), Next do [TryHackMe BufferOverflow Prep](https://tryhackme.com/room/bufferoverflowprep) room following [this video](https://www.youtube.com/watch?v=1X2JGF_9JGM) from Tib3rius.
* For Linux and Windows Enum/Privesc, there's no alternative than practicing vulnerable machines yourself and gaining experience. As you may already have heard of TJ_Null's [OSCP like boxes list](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=1839402159), do those, and after you are done with a box read writeup for that box from [0xdf's blog](https://0xdf.gitlab.io/) and watch [Ippsec's video](https://www.youtube.com/channel/UCa6eh7gCkpPo5XXUDfygQQA) on that too. Create a habit of writing a short walkthrough/report as you go on with your hacking. This will help you with your report writing skills and also you will have your own private database of experiences you faced before so you can just grep from the parent directory of all the boxes you have done if you think you faced something similar before! Use this [search tool](https://ippsec.rocks/) to find any specific tool/method used by Ippsec on a machine.
* Don't shy away from using Metasploit! If you encounter an exploit that is doable both manually and using Metasploit, practice both. Don't shy away from using your One-time Metasploit usage in the exam either but I'll suggest using it at the very end when you think your exploit will work!
* Guess check try! repeat this for anything that doesn't look normal during enumeration. If you are making no progress for a certain amount of time in HTB or Labs, get a hint for that part. Instead of reading to writeup to confirm if your guess was correct first try yourself. Once you have some experience you will be able to differentiate between what's a rabbit hole and what's a potential exploit path!
* Join [Infosec Prep Discord](https://discord.gg/mEtEFhp)! you'll find a bunch of good people there preparing for many certifications and helping each other.
* Lastly, borrowing the advice from someone which helped me as well, just because you did less boxes than someone else and he failed his exam don't loose your confidence because of it! What matters most is how fast you as a person learn and absorb new concepts!

I wish you all the best in your journey!

