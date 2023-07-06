# Introduction

## [OpenProject](https://www.openproject.org/) on a [RaspberryPi](https://www.amazon.com/Raspberry-Model-2019-Quad-Bluetooth/dp/B07TC2BK1X/ref=sxin_3_ac_d_pm?ac_md=3-0-VW5kZXIgJDc1-ac_d_pm&cv_ct_cx=raspberry+pi+4+4gb&keywords=raspberry+pi+4+4gb&pd_rd_i=B07TC2BK1X&pd_rd_r=e622118f-8bbf-43e5-ba7b-2da482551e9b&pd_rd_w=30pUf&pd_rd_wg=8t2FQ&pf_rd_p=0e223c60-bcf8-4663-98f3-da892fbd4372&pf_rd_r=8PG76Q9PBRF32SW9MH8G&psc=1&qid=1582422684)-based server 

[OpenProject](https://www.openproject.org/) is a fully featured, open source project management toolbox.  
A free community version has been released, but some premium features are limited to the paid-for cloud & enterprise versions. 
If you want to run your own server, the project suggests a Linux-based system with 4 GB RAM, and supplies various .deb/.rpm packages, or docker images for easy installation. 

The newer Raspberry Pi 4 boards are compact and power efficient, and with 4 cores & 4 GB of RAM at least on paper look capable of running OpenProject. This would make for a very power-efficient and compact solution for (at least) small teams or local test installations.  

The problem? OpenProject officially does not support the ARM architecture, so any described installation methods based on .deb/.rpm packages or docker fail miserably. The support forum has a few threads on this dating back several years, but little to no help has been forthcoming to to make this happen. But if you found this page, your probably know all about that already. 

## Good news: OpenProject runs on ARM (unofficially, at least)!

It's possible to get OpenProject working on a Raspberry Pi. I highly recommend a RPi4 with 4 GB RAM (or a similar board). It might be possible to get by with the 2 GB version and sufficient swap space, but that would be untested as of now, and seems ill advised. During the installation and compilation process, memory usage maxes out at about 3.1 GB. 

## I just want to give this a go quickly - is there a ready-made system image?

There is an image created by madewhatnow, but it is not recommended because it is from 2021 and has many security risks.

madewhatnow prepared a Raspian lite-based image in Feb 2021, which should get anyone up and running in half an hour, but has some security issues, unless passwords are changes, etc.

The image file is [here](https://drive.google.com/file/d/1qBzWME8BCVja0HickLo_SOcvsUL5sUEC/view?usp=sharing), around 12gb and should fit any 16gb+ memory card. It does require a RPi4, with ideally 2 or 4gb memory. SSH is enabled (pi//raspberry), OpenProject is still on the default login (admin//admin), and you will have to enable wifi (/etc/wpa-supplicant/wpa-supplicant) or ethernet to connect to the system. Then set up email notificatiosns in OpenProject (see below). 

Please report problems and successes to the original repository.

## Status (Feb 2021)

I original wrote the instructions in February 2020, for OpenProject 10 - and while they (mostly) worked, they were still somewhat buggy in an unpredictable way. I have received a surprising amount of emails asking for the system image, or various fixes, and finally revisited the protocol in 2021. OpenProject recently released version 11 - and somewhat surprisingly, in 2021 the process is a lot smoother. Starting with a Raspian Lite image on a  RPi 4, the whole process was done in a few hours, with minimal issues. The protocol below is updated to reflect the required changes. 

There is an ongoing discussion in the OpenProject forum regarding tweaks and fixes: https://community.openproject.org/topics/6873
