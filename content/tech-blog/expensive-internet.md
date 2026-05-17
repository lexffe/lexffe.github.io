+++
title = "My internet is too expensive so I am test-running 5G home internet."
description = "Moving house is on the horizon, yet my ISP contract just expired and went into a rolling plan. Can I find a way to lower my bills?"
date = 2026-05-17
+++

Note: while this article is not generated or written with LLM/AI, I have used LLM/AI heavily for the research and implementation. Reader’s discretion is advised.

# Introduction

2 months ago I received an email from Hyperoptic notifying me that my 12-month contract has ended and they are putting me on the rolling contract. The commitment was £35 a month and the rolling contract costs £45 a month - a £10 increase. I kept using it for 2 more months until I realised that paying this much for internet access is just not financially responsible.

I can sign another 12 month contract to get the price down, but *moving is on the horizon*. I am thinking of moving in the next 6 months and if the new place doesn’t have a Hyperoptic connection then I would have to pay an early termination fee. If the new place has a better broadband deal with another company, I would not be able to switch over. 

Naturally, people would just look for a new internet provider and do their research to get the best deal. ISPs like BT or Virgin Media tend to have coverage almost everywhere in the country so signing with them shouldn’t cause any issue down the road. But apparently the building I live in is exclusively on Hyperoptic. I cannot find any other ISP that provides fibre services in my building, and I refuse to use xDSL in 2026\.

Here’s what the Hyperoptic connection gave me:

- Advertised *symmetric* 150Mbps Fibre To … something?  
  - I am pretty sure it is fibre to the premise (FTTP). There is only an Ethernet jack in my flat, yet Hyperoptic offers Gigabit “fibre” so I assume there is an ONT and router nearby in the building that terminates, media conversion and rate limiting, and I assume they can *only* offer Gigabit fibre because the last mile/meter is a cat.5e or cat.6 cable.   
  - The real speed is actually 160 up / 140 down.  
- From my UniFi Dream Router (UDR), the ping latency to CloudFlare is a consistent 6ms.   
- A CGNAT IP. I don’t get my own IPv4 address, which is meh but understandable. They offer a static IP for £5 extra a month and I did think about it initially but I don’t see the point anymore after I started using Tailscale.  
- A /56 block. This is standard on most broadband these days.

I replaced the ISP router with my own. A MAC address cloning worked. Everything has been working nicely since.

## The user profile

Apart from watching YouTube and listening to music and scrolling on social media, I need to pull/push docker containers, video telephony across the world, access my home NAS through SMB/NFS/SSH/VNC, which high bandwidth and low ping would help.
My ISP also gave me a /56 block, so I assigned a /112 to Kubernetes services and a /64 to pods on the k3s single node on my NAS. 

## The haggling

I messaged Hyperoptic just to see if they could get me a better deal (even better deal than my original 12 month plan).

For curiosity reasons, I ~asked~ prompted Gemini to ~find~ aggregate the average prices of 150Mb broadband across the country and the response was that in the UK, the average was £22 to £26 a month. They quoted me a £40 12 month contract, which is more expensive than what I see on broadbandchoices, uSwitch and what they offered right before the commitment ended.
I questioned them and they said everyone gets one introductory deal, and renewal costs the same for everyone. They also said that the offer in the “renew your contract” email was valid only until the end of the 12 month commitment.

I did not like that a single bit. I was already paying more than average before and now they want to charge me 1.5x avg, so I wrote a fairly strong-worded response:

> £40 for 150M is nearly 1.8x the national average and I cannot justify nor support the monopoly. If your retention team could offer a better deal for a 12 month 150Mb then I would consider it, otherwise consider this as my formal 30 days notice for disconnection. I will switch to 5G instead.

They came back with 

- Gigabit: “39.99.99”/month for 12 months, then 63/month rolling,
- 500Mb: 37.99/month for 12 months, then 53/month rolling, and  
- 150Mb: 35.99/month for 12 months, then 45/month rolling.

They saw me paying 45 for 2 months and think that they can get 40 with a pretty Gigabit badge because people always want faster speed, non?
OK I was not being fair or nice here. They have a job to do just like us and they are just humans. Still, I don’t think those are good deals at all.

# Solutions?

So what are my solutions, apart from being tied to Hyperoptic for another year? Based on what I know and what Gemini confirmed, I have two choices: 

## Starlink

I have had super good experience with Starlink, despite not having used their services directly. I was on a flight to Hong Kong with a stop in Qatar and Qatar’s Manchester to Doha leg has Starlink onboard, as well as the return flight. The ping was sub 60ms and I was watching 1080p YouTube on the flight and calling my family *on the flight*. I thought “about time. It’s 2026\. We should have had fast internet on flights ages ago.”

That said, I am not getting Starlink. Starlink costs £35 and only gives me 100Mb. Even though I have a balcony, there is no guarantee that I can receive a signal from the satellites.

## 5G

This fits most of my criteria, notably the flexibility to get internet relatively cheap. But two big problems remain: latency and bandwidth.

# The standalone problem

5G Standalone (branded as 5G+ and 5G Ultra) is a 5G deployment mode that does not depend on any existing 4G/LTE infrastructure. My phone never receives a Standalone signal and so I could never test the performances properly. A quick Google search did tell me that being on a MVNO means you are deprioritised on the Standalone adoption list. Some suggest that VOXI (Vodafone’s MVNO) can sometimes catch a Standalone signal with IPv6 addresses but I have not tested yet. How about going to the MNOs directly?

Apparently EE, O2 and Vodafone all have 5G Standalone available in my area, and it is available “indoor”. I am taking this with a few grains of salt though.

The only reason I care is because of latency. Wireless just doesn’t beat fibre optic latency (unless you are using W60G). Even Starlink has better latency than 5G. My hope is that Standalone improves the latency because the channels are less crowded. Therefore, here is a question to myself: can I live with “slightly higher” ping?  Can I live with the jitter?

(Note: my past experience and expectations here have been proven completely wrong. In the next article I go deep on testing with latency/ping testing on Non-standalone deployments.)

# The expensive 5G modem problem

5G modems and gateways are actually quite expensive. The new Unifi 5G WAN costs £250 pre VAT, and GL.iNET’s travel 5G router costs £150. Both Vodafone and EE give you Customer Premises Equipment (CPE) when you subscribe to their 5G home broadband plans. I want to use my own router and I don’t think any other CPEs are hackable-enough to be set to a “bridge mode”. 

(Note: I was partly wrong here. Vodafone’s CPE lets you set it to “IP passthrough mode” and I got an IP from Vodafone: one from the 10.0.0.0/8 block.)

Enter the toy drawer, an assortment of techs sitting in the drawer not being used actively. We have

- a Samsung Galaxy Z Flip 5,  
- a Samsung Galaxy Tab S8 Ultra,  
- an iPhone 12 mini, and  
- a MikroTik hAP ac2.

The iPhone 12 mini has a fairly dated modem and using USB tethering is not feasible because we are still limited to USB2 speed. Tethering (repeating) through Wi-Fi is just not a good idea. Using Wi-Fi tethering might work, but that would mean that I lose control over routing and firewall. Gemini reminded me that I have a phone with a USB-C port, and there are USB-C to Ethernet adapters with PD passthrough. In theory, I can hack together an Android + MikroTik setup.

# The IPv6 problem

I use IPv6 a lot on a daily basis. Tailscale works the best when both peers have IPv6 connectivity.   
GitHub still doesn’t have IPv6 though.

Most mobile network operators in the UK are still using Carrier-grade NAT (CGNAT) without IPv6 connectivity. By going through the phone’s NAT and CGNAT, we are adding a few more milliseconds of latency on an already supposedly slow network. Not good.

The only player I know of on the UK mobile network market that has native IPv6 is EE. They are an early adopter to the 464XLAT transition mechanism. BT did a presentation on EE’s IPv6 setup back in 2018\. ([Link](https://www.ipv6.org.uk/wp-content/uploads/2018/11/Nick-Heatley_BT_EE_Update_UKv6Council_201801207.pdf)) There’s also another deck of slides I found quite interesting on 464XLAT by the IPv6 Company at RIPE (that’s a thing???) [Link](https://www.ripe.net/media/documents/Jordi_Palet_-_464XLAT-for-millions-v3.pdf).

I prompted Gemini for solutions on getting IPv6 to my client devices and I was recommended using NAT66 or ND proxy. More on that later.

# The next month

We have a matrix:

| Features\\ISP | EE | O2 | Vodafone core | Three core | MVNOs |
| :---- | :---- | :---- | :---- | :---- | :---- |
| 5G Standalone | Yes on some plans | Yes if you pay monthly | Yes on some plans | No | Mostly no |
| Native IPv6 | Yes | No | Yes on Standalone supposedly | Yes | Mostly no |

5G services costs just about the same if not cheaper than what Hyperoptic offered, with the opportunity that you may get more than 150Mb on Standalone, not to mention the portability of the setup. In fact, if you are not a power user like me, you can already get an unlimited SIM sometimes for as low as £16 and skip all this faff.

I got myself an EE unlimited data SIM card. I am going to put it into my Z Flip 5 and see if I can catch a 5G Standalone signal. I also ordered a 2.5Gb Ethernet to USB-C with PD adapter on Amazon. I will experiment with IPv6 as well and see how well this phone+router setup fares.

# The alternative outcome if 5G is not good enough

Even if 5G home broadband is not good enough, I will still be learning something new. I have been taking a lot of photos and videos and the upload speed to Google Photos is kind of slow.

I found this Rust tool that lets you load balance over multiple connections. It is not a reliable and scalable solution (what if I am using other computers at home?). However, this learning experience lays the groundwork for multi-WAN, and I can potentially put my underutilised phone plan to use when I need to do large uploads to boost the speed.

# Appendix: A\&A’s raw L2TP tunnel

I came across [this post](https://www.ispreview.co.uk/talk/threads/looking-for-router-with-464xlat-support.44091/) on ispreview. Someone mentioned the Andrews & Arnold L2TP service. It is the first time I have seen ISPs offering static IPv4+IPv6 over raw L2TP (NOT VPN) to *consumers*. On mobile plans where there is no IPv6, you can still get IPv6 by subscribing to this for £10/month.

# Appendix: Other random thoughts in my head

- If I go away for weeks from home, I can switch the SIM contract to a lower price and lower data cap one, which can save me some money. (Assuming I don’t download/upload large amounts of data from my NAS.)  
- The router is currently situated near the doorway because that’s where the engineer terminated the ethernet.   
  - My Optiplex NAS has to sit next to the router.  
  - I can only use homeplug to get ethernet to the rest of the house. This adds quite a bit of latency AND I have had issues where the BT homeplug devices would slow down for no reason and I would have to power cycle it.  
  - An unintentional consequence is that I can finally move the router and NAS into the study, where most of my devices are, and I can have a direct connection between all my devices and the router, which speeds up NFS/SMB/VNC/SSH/whatever.
