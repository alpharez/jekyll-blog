---
layout: post
title:  "Adventures with stunnel"
date:   2022-02-07 06:11:53 -0500
categories: unix
---

![appslow](/assets/images/ziqpaz94gm8a1.jpg)

I was tasked with figuring out a strange issue.  A web service kept going unavailable for 5-15 minutes at a time.  These outages would occur once per day for 1-3 days and then a month or two later it would happen all over again.  It was impossible to reproduce.  I had to figure this out.

Was it the Cisco ASA firewall with a firepower intrusion prevention module or the F5 load balancer?  Was it a layer 3 issue, packet loss or network congestion?

We looked at the routers and switches and found no errors on any ports and no congestion.  The firewall did not show any dropped packets.  We setup packet captures on the web servers and the firewall hoping to catch the outage in progress.  Eventually tthe outage re-ocurred and we were able to view a packet capture.

The packet captures showed the outage as a period of lower than normal throughput between clients and the web server, with a large spike at the end of the outage as clients reconnected.  We also noticed increased TCP resets during the outage and right after.

Not finding any obvious network issues, I started looking at the application itself.  The web service did not do any encryption, so stunnel was put in front as a SSL/TLS/VPN proxy.  Encrypted traffic would terminate on stunnel from the clients, and then the web service would get the clear text.

I wanted to look at the log files that stunnel created, and in the server messages log there were a lot of stunnel logs.  stunnel was setup to log everything it did, so there were gigabytes worth of logs to look through.  I was able to pinpoint the exact start and end time of the outages in these logs, and I could see in this period there was nothing logged.  stunnel was completely frozen during these outages.

I believe the way stunnel was setup was not optimal for this application.  Some input queue was filling up and it took a few minutes to straighten everything out, flush queues and start up connections again.  Instead of trying to setup stunnel more optimally, it was decided to use another encryption method.

Moral of the story here is what I always recommend when I face a tough problem like this: have the whole team look at every piece in parallel, don't just *blame the network*, look at the servers, look at the application, look at the client application.  Have everyone look at everything.  This particular case took a long time to figure out because the server people and the application people were waiting around for the network guys to fix it.
