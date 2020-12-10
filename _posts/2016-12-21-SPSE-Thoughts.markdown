---
title:  "SPSE Certification"
date:   2016-12-21 10:00:00
categories: [SPSE]
tags: [SPSE, Python, SecurityTube Python Scripting Expert]
---

The [SecurityTube Python Scripting Expert (SPSE) course](http://www.securitytube-training.com/online-courses/securitytube-python-scripting-expert/index.html){:target="_blank"} is an online certification, and accompanying videos designed to teach students how to apply Python programming techniques to security and networking concepts.
<p align="center">
<img src="{{ "images/SPSE/PYTHON.png" | prepend: site.baseurl }}">
</p>

### Thoughts on the Course ###

This course is actually something my previous employer bought me a couple of years ago, that I put a little bit of time into, but haven’t really looked at in a long time. One of the positives of this certification and course is you can do it entirely at your own pace. All the materials are DRM-free and downloadable, and you don’t lose access to the course website after a certain period of time. I made the decision in November to attempt to complete the SPSE coursework and certification. A negative of adding this cert to your LinkedIn is the name. "SecurityTube" is a horrible name, and sounds like some sort of site for hosting Rule 34 DEFCON videos. Vivek also runs [PentesterAcademy](http://www.pentesteracademy.com/){:target="_blank"} which is a much better name, and would be a better title for the certification.

I wouldn’t recommend this course to someone that isn’t somewhat familiar with Python as well as the various concepts outline in the course. Vivek does fairly minimal explanations of background concepts, which is fine if you’re somewhat familiar with them already, but I think it would be difficult for a beginner to understand. The course requires a fair bit of extra research, during which I have learned/re-learned a lot of important concepts, but be warned that you’ll have to do this kind of extra work with little guidance. The course should point students towards helpful resources to help focus their study.

Another recurring downside is that it hasn’t been maintained or kept up to date. This seems to be a theme with Vivek’s courses as the [iOS Security Expert](http://www.securitytube-training.com/online-courses/securitytube-ios-security-expert/){:target="_blank"} course lists iOS 5.1 as the minimum requirement. This course is plagued by outdated modules which have been replaced by better alternatives, which I’ll cover on a per Module basis. 

Another problem is its lack of testing framework, skeleton code, or instructions on verifying your code is working correctly. For example, an exercise will instruct you to inject ARP packets but not tell you how to verify that things are working correctly. You can view the exercise solution videos, but those give away too much information. Trying to learn the commands for ARP at the same time as writing a script to manipulate ARP traffic is asking a novice/intermediate to solve too many problems at once. Is my script wrong? or are my tests wrong? I tried to document helpful resources on a per-exercise basis in the code comments.

Another issue I take with this course is its lack of adherence to a Python style guide and it's lack of compatibility between Python 2 and 3. Fairly minor gripes—but I think that if you’re instructing students, you should setting a good example.

I’ll be periodically updating this post as I complete modules, and sharing my thoughts as I go.

### Module 1 ###

I don’t have much to say about Module 1. I reviewed the PDF slide notes and exercises and determined that is was largely material that I already knew. I think it’s too basic for someone with a background in Python, but not nearly enough for someone that isn’t familiar with Python. I would strong recommend starting with a different course if your Python skills are minimal.

### Module 2 ###
Module 2 is great at times, and confusing at others. After completely Modules 2 and 3 it’s clear that the course was originally in a different order. Module 2’s system programming aspects have good exercises, like visually representing a directory traversal and connecting to FTP sites. Unfortunately there is a bunch of other stuff that is thrown in haphazardly and requires you to complete Module 3 prior to completing the exercises. 

There is no shortage of things that fall under “System Programming” that could be included in this module.

[Here are my solutions on GitHub.](https://github.com/leifdreizler/SPSE/tree/master/Module%202){:target="_blank"}

### Module 3 ###
Module 3 has a lot of overlap with Module 2 exercises. Many of the exercises center around adding multiprocessing or multithreading to things, despite this being a Module 2 concept. I like the idea of integrating concepts from previous modules into future exercises, but they shouldn't be the center of the exercise, or too reptitive. Module 3 should focus on sockets and the Scapy module.

The sockets work is interesting and helps you learn what’s going on beneath the hood with Scapy. One problem I have with the sockets programming exercises are that they rely heavily on “magic numbers” which aren’t explained or referenced anywhere in the exercises. There is value in leaving certain things to the student to discover, I don’t think random static ARP packet field values are in that list.

The Scapy exercises are all pretty good with the exception of the Wifi Sniffer exercise. Getting Wifi working in a Linux VM is actually easiest if you have a USB wifi dongle, which most people don’t have. If I had known about the [Python3 version of Scapy](https://github.com/phaethon/scapy), I would have checked it out. All of the Scapy related exercises are completed using Python2.

[Here are my solutions on GitHub.](https://github.com/leifdreizler/SPSE/tree/master/Module%203){:target="_blank"}

### Module 4 ###
Most of the libraries used in Module 4 have been replaced by better alternatives. I’m not promoting chasing the hottest new module/framework/etc. but not mentioning the [Requests](http://docs.python-requests.org/en/master/){:target="_blank"} library is criminal. It’s one of the most popular Python packages of all time, is beautifully written, and frequently mentioned in StackOverflow posts. urllib should get a brief mention (similar to sockets before Scapy), but Requests should receive most of the attention.

Mechanize is also showing its age, with the last commit on GitHub being in 2012 and having no support for Python3. I would recommend replacement with MechanicalSoup, which is built on top of BeautifulSoup and Requests. BeautifulSoup gets covered in the current coursework, and Requests should be, making MechanicalSoup a logical progression. In another module you're told to go research ZSI, a Python library that hasn't been updated in almost a decade.

The last two video lectures combine attempt to combine content from the previous videos and combine it into an exercise that reviews previous material. Unfortunately, it also introduces storing information in databases which hasn't been covered at all and the remainder of the OWASP Top 10 in two videos with a combined runtime of ~5 minutes. I took a databases course while studying abroad in college 5 years ago, and did a couple years of appsec consulting so I was reasonably prepared for this exercise, but many students wouldn't be based off of the previous coursework.

Again, I'm all for encouraging students to do their own research, but there should have been at least an introduction to databases before giving an exercise that relies heavily on using one.

[Here are my solutions on GiHub.](https://github.com/leifdreizler/SPSE/tree/master/Module%204){:target="_blank"}
