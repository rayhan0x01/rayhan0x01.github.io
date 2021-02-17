---
layout: post
title:  "National Cyber Drill 2020 Forensic challenges writeup"
date:   2020-12-15
categories: CTF
thumbnail: /img/ncd2020.webp
tags: ctf,ncd2020,autopsy,forensic
---
Two days ago, I collaborated with few students like myself from "The infinity bytes" and participated in the first National Cyber Drill 2020 organized by the Bangladesh Government's e-Government Computer Incident Response Team (BGD e-GOV CIRT) and secured 2nd place against 234 teams.

This contest had teams comprised of cybersecurity personnel from financial and non-financial institutions, regulatory bodies, corporations, law enforcement, universities, and IT organizations of Bangladesh. The challenges were mostly blue team oriented which not really is my forte as I like red team challenges more. So here goes my writeup for the challenges in the "Forensic" section where I first time used [Autopsy](https://www.autopsy.com/download/) to find flags scattered inside an "Encased Image file".

The Linux version of [Autopsy](https://www.autopsy.com/download/) has less features than the windows version so I installed the latest version available for windows, created a new case, added the `hdd1.e01` file as data source.

&nbsp;

## # Find the hidden Treasury : 50

Task: Find the file with Japanese text and mention the Modified time of this file from the file.

Using the "Keyword Search" feature, searched for the word "japanese" and found the target file, last modified time was the flag for this one.

![forensic1](/img/forensic1.png#center)

&nbsp;

## # Contraband is illegal. : 50

Task: What is the name of the person, who was trying to sell something illegal to user WES? 

The full name of user WES is "Wes Mantooth". Looking into the "Email Messages" section, found an email copy where a person named "John Washer" was trying to sell him something. Sender's name was the flag for this one.

![forensic2](/img/forensic2.png#center)

&nbsp;

## # Evidence is the final conclusion ...right? : 100

Task: What is the evidence file name which contains info of email address robin.keir@foundstone.com?

Simply searching the email address via keyword search lead to the binary file `trout.exe` which was the flag for this one.

![forensic3](/img/forensic3.png#center)

&nbsp;

## # Shutdown is the best policy to ensure security. : 100

Task: Mention the OS last shutdown time by the user WES.

The "Operating System User Account" section had the "Date Accessed" field for the user Wes Mantooth which turned out to be the flag in the format `DD-MON-YY`.

![forensic4](/img/forensic4.png#center)

&nbsp;

## # SID is a unique value... what do you think? : 100

Task: What is the SID of the windows user Dracula in this system?

The "Operating System User Account" section had the "User ID" field for the user Dracula which was the flag for this one.

![forensic5](/img/forensic5.png#center)

&nbsp;

## # SECURITY IS THE FIRST PRIORITY : 100

Task: What is the user name used in the Yahoo application on this system by user WES?

The `\img_hdd1.E01\Program Files\Yahoo!\Messenger\Profiles\` contained the folder name "incisorman420" which was the flag for this one.

![forensic6](/img/forensic6.png#center)

&nbsp;

## # Have you taken your Breakfast? : 100

Task: Is there any mention of any breakfast item in this evidence? Mention the file name from the following.

Found `pf3.wav` file in the user Wes Mantooth's Music directory which contained a dialogue from the movie "Pulp Fiction", "Looks like me and Vincent caught you boys at breakfast. Sorry about that. Whatcha having?". The flag was the audio filename for this one.

![forensic7](/img/forensic7.png#center)

&nbsp;

## # Hiding information is important for Security : 100

Task: Find the difference between these mountain1. and mountain2 image.

For this challenge we were given two identical bitmap images "mountain1.bmp" and "mountain2.bmp". Other than their visual difference being just a red dot, extracting their strings and comparing them via `diff` showed few character difference. After a few tries the flag turned out to be "eyes".

![forensic8](/img/forensic8.png#center)

&nbsp;

## # Hidden Museum : 100

Task Please extract hidden text from this image. The passphrase is the previous simple forensic 001 flag.

The "simple forensic 001" just had a nested zip file where the flag was written inside an image with value "ACNESTICS". For this challenge a different image was given of a museum. Since the task mentioned passphrase and the password was already in hand, checked with steghide and found an embedded file which contained the flag.

![forensic9](/img/forensic9.png#center)

&nbsp;

## # Let's watch the TED TALK : 150

Task: You are requested to find the flag from attached file.

The file given was an mp4 video of Bill Gate's Ted Talk about "The next outbreak? We're not ready".

The flag was scattered in pieces throughout the video, combining the flag resulted in "{Even Though We Consider The Distance,We Are Still Connected}".

&nbsp;
&nbsp;

There were two other challenges related to Autopsy that we didn't solve. Overall this drill was a good experience getting to enjoy blue team operations and other challenges like incident handling, networking were also fun to solve. Thanks to BGD e-GOV CIRT for organizing a great event like this where we were able to compete against teams comprised of all sorts of institutions in Bangladesh!

&nbsp;