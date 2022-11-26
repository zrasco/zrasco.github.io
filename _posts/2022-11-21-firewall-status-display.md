---
layout: post
title: 
date: 2022-11-21 16:36
category: 
author: Zeb Rasco
tags: [iptables, firewall]
summary: 
---

# Firewall Status Display
[![](/assets/2022-11-21-firewall-status-display/2022-11-22-21-17-01.png)](/assets/2022-11-21-firewall-status-display/2022-11-22-21-17-01.png)
[![](/assets/2022-11-21-firewall-status-display/2022-11-22-21-14-00.png)](/assets/2022-11-21-firewall-status-display/2022-11-22-21-14-00.png)
*For my Cybersecurity MS Practicum Project (CS6747) at The Georgia Institute of Technology.
Author: Zeb Rasco.
Date: 11/22/2022*

## Background
Home routers are a prevalent part of today's internet ecosystem. Home internet packages from ISPs typically have a router included, or a router/modem combination. A typical home user will have multiple devices which need internet access, such as smart TV's, phones, tablets, computers, and various Internet of Things (IoT) devices such as cameras and smart appliances. A router is required for multiple devices to connect to a single internet connection, and virtually all modern routers also include a wireless access point for wireless devices. This direction technology has taken has resulted in a very large amount of routers existing in the world today, which serve as the bridge between a private home network and the public internet.

IoT devices are prevalent in today's world with an estimated 11.28 billion devices in 2021 and are projected to almost triple to 29.42 billion devices by 2030 *(Vailshery, IOT connected devices worldwide 2019-2030 2022)*. In addition, IoT devices often run on proprietary embedded systems and vulnerabilities which are difficult to patch, if a patch is even available at all *(Zou, IOT devices are hard to patch: Here's why-and how to deal with security 2022)*.

[![](/assets/2022-11-21-firewall-status-display/2022-11-22-18-37-33.png)](/assets/2022-11-21-firewall-status-display/2022-11-22-18-37-33.png)
<sub><sup>Fig. 1 - Projected growth of IoT devices over the next 10 years (Vailshery, 2022).</sup></sub>

Because of this threat, routers are often hardened with a firewall component. While these built-in firewalls provide adequate protection for many uses cases, attacks continue to become more sophisticated and devices continue to become infected with various types of malware, giving the attacker access or even control of the entire home network.

With our background discussion of routers and firewalls complete, we can now discuss the problem at hand.

## Problem statement

Firewalls which come bundled with home routers are often proprietary and lack a consistent interface and consistent reporting/features. While these routers vary in their effectiveness to protect against common attacks, often the user is unaware of the overall threat environment. For example, a user using stock equipment is unlikely to be aware of a targeted attack from a determined adversary.

Intermediate to advanced users will sometimes install custom firmware on their existing routers, including DD-WRT, OpenWRT, Tomato, and others. This involves replacing the existing stock firmware with a more feature-rich firmware which will typically include SSH access, an enhanced web interface, and other value-adds. These variations of firmware will typically a firewall which can be managed with the iptables command line tool and therefore the firewalls are highly customizable. However, while this does provide enhanced protection this does not solve the issue of lack of insight into threats. The user is typically only able to use a command to view raw firewall data in the form of Linux-style syslog/dmesg entries.

To deal with this shortcoming, such a user is able to install orchestration and/or logging software which may contain dashboards showing a more insightful picture into the threats their network may face. However, such software typically requires a substantial investment of infrastructure, as we will explore in-depth later. In addition to the hardware requirements, there is also a large degree of setup and customization required, taking hours or perhaps days to fully set up. These solutions typically also require a certain degree of maintenance in the form of Virtual/Physical Machine resource management, operating system updates, application updates, further customization after the updates, incompatibilities with newer updated libraries, and so forth. These updates are sometimes complex and may also fail, which in turn increases the technical burden demand on the user.

One alternative approach is to invest in a Small Form Factor device which then has a purpose-built OS and software suite/IPS installed such as the acclaimed IPFire solution. However, this has an obvious infrastructure cost as the base system itself with hardware-supported AES encryption (for the Wifi network, etc) and multiple network ports can easily run 200 to 300 USD. While such a solution is indeed effective, it does require that infrastructure cost that many users may not be ready or able to afford at any given time.

I therefore propose there is a problem (and niche) for users who find themselves in this situation and cannot afford either the infrastructure costs and/or the time required to set up a fully orchestrated solution. This demographic of users are aware that they have a need to gain insight into network attacks but want to do so with relative ease until they may perhaps find themselves in a position to implement a more comprehensive orchestration framework.

[![](/assets/2022-11-21-firewall-status-display/2022-11-22-21-04-21.png)](/assets/2022-11-21-firewall-status-display/2022-11-22-21-04-21.png)
<sub><sup>Fig. 2 - Example SFF firewall device (Vnopn Micro Firewall Appliance) with WiFi antennas. Current cost is 262 USD.</sup></sub>

## Solution

Considering the relatively onerous requirements of the solutions above, my proposed solution for this niche problem is a new iteration of an application I previously developed for this purpose. I have a created an app with the working title of Firewall Status Display. This app is designed to quickly provide useful insights into router firewall activity, with minimal setup and infrastructure requirements. It runs on Windows 10, which has over a 71% market share of all desktop systems (Desktop windows version market share worldwide 2022).

### Architecture and implementation details

This application was written using Visual Studio 2022 Community Edition with the .NET 6 SDK and also uses Progress Telerik for WPF UI controls. It uses a standard MVVM design pattern and dependency injection to share services amongst program components without the need for static dependencies. I tried to be as by-the-book as possible with this to ensure the program runs as reliably as possible.

The program runs as an application on common Windows desktop systems and listens for firewall data on UDP port 514. These firewall UDP datagrams are generated from a remote syslog daemon running on a Linux machine, or custom firmware on a router. These datagrams are commonly used iptables log output entries, which are typically output to the system log on these devices and can be optionally redirected to a remote server, in this case, the machine running the firewall status display.

### Design goals & pre-requisites
The design goals of my solution are straightforward:
1) Easy and quick setup
2) Minimal requirements
3) Minimal configuration
4) Feature-rich insights out-of-the-box

The application is portable by default and can simply be decompressed and executed. The pre-requisites for installation are:
- .NET 6 Runtime
- SQL Server Express (I used 2019 but any should work)
- 150MB disk space

Thus with the above facts design goals #1 and #2 are fulfilled. Once in place, a user needs only to edit the appsettings.json file in the application directory to point to the database (usually localhost). The user then need only to run the application and use the Geolocation DB import tool to import a free Geolocation database. Finally, the user configures their firewall to output iptables entries to syslog (usually on by default), and then configures their firewall to output syslog entries to the client machine. These straightforward steps seek to fulfill design goal #3.

Finally, the user will immediately see several useful tabs which seek to fufill design goal #4. They can immediately see a filterable/sortable table of dropped firewall packets, graphs by country and port, port scans, and general firewall statistics. These statistics include average packets per day and show a graph of overall activity over time.

### Availability to the general public
The application is open-source and available on my GitHub repository: https://github.com/zrasco/firewallstatusdisplay

My goal with this project was to give a viable security tool back to the community, and release the source code to encourage others to add onto the functionality. Many others have helped me to develop my skills throughout my career and I am happy to return the favor by releasing this application under an open-source license. It is free to use for personal, educational, research, and commercial purposes (though it uses Progress Telerik so you may need that license) with no attribution required.

In addition to the source code being available, I still plan to continue work on this application into 2023. One of my first objectives is to set up a portable and MSI installation package as well as a capability of running the program whenever Windows starts, the goal of which being to ensure consistent recording of firewall data.

### Features
We'll now review the main features of the application. The application has 4 main areas:
- Status
- Syslog
- Firewall log
- Settings

Each of these application areas can be navigated by clicking the corresponding navigation links on the left hand side of the application. The relevant sections will then appear on the right. Users can further drill down into areas of interest by using the tab controls embedded in the main layout pane on the right-hand side.

#### Status dashboard - Traffic by country

This is the default view of the program and will group dropped firewall packets by their country of origin. There is a conveniently placed sortable and filterable datagrid to view this information in more detail.

[![](/assets/2022-11-21-firewall-status-display/2022-11-23-20-33-07.png)](/assets/2022-11-21-firewall-status-display/2022-11-23-20-33-07.png)

#### Status dashboard - Traffic by port

This layout is similar to the previous section, but instead shows traffic by port number. This is useful for identifying common attack vectors used by adversaries.

[![](/assets/2022-11-21-firewall-status-display/2022-11-23-20-59-07.png)](/assets/2022-11-21-firewall-status-display/2022-11-23-20-59-07.png)

#### Status dashboard - Port scans

As in the previous 2 layouts, port scan activity is available in the accompanying data grid. There is also a chart showing trends of port scan activity over time, where spikes may possibly indicate a coordinated attack from a focused adversary instead of a random sweep from an adversary scanning swaths of IP addresses. When a port scan is detected, it is added to the list and an alert is given to the user via the notification feature native to Windows 10.

[![](/assets/2022-11-21-firewall-status-display/2022-11-23-21-00-53.png)](/assets/2022-11-21-firewall-status-display/2022-11-23-21-00-53.png)
[![](/assets/2022-11-21-firewall-status-display/2022-11-24-10-18-49.png)](/assets/2022-11-21-firewall-status-display/2022-11-24-10-18-49.png)

#### Status dashboard - Statistics

This last pane of the status dashboard shows general firewall statistics. IP geolocation information utilizes an LRU cache for fast lookups. As you can see from the output I have a lot of days worth of data and do plan on cleaning this up...

[![](/assets/2022-11-21-firewall-status-display/2022-11-23-21-35-02.png)](/assets/2022-11-21-firewall-status-display/2022-11-23-21-35-02.png)


#### Syslog output

This section of the application shows the raw syslog output. This can be useful for debugging purposes and can also be used to check non-iptables related logs, if needed.

[![](/assets/2022-11-21-firewall-status-display/2022-11-24-10-03-50.png)](/assets/2022-11-21-firewall-status-display/2022-11-24-10-03-50.png)

#### Firewall log

Here we can see the actual firewall log entries arranged in a data grid. This grid allows filtering and sorting, which enables the user to quickly drill down into specific information.

[![](/assets/2022-11-21-firewall-status-display/2022-11-24-10-08-16.png)](/assets/2022-11-21-firewall-status-display/2022-11-24-10-08-16.png)
[![](/assets/2022-11-21-firewall-status-display/2022-11-24-10-09-05.png)](/assets/2022-11-21-firewall-status-display/2022-11-24-10-09-05.png)

#### Settings

Finally, in the settings area we can see the top part of the layout contains application-specific information. The user can see the status/health of the application and take action necessary.

[![](/assets/2022-11-21-firewall-status-display/2022-11-24-10-10-17.png)](/assets/2022-11-21-firewall-status-display/2022-11-24-10-10-17.png)

[![](/assets/2022-11-21-firewall-status-display/2022-11-24-10-10-52.png)](/assets/2022-11-21-firewall-status-display/2022-11-24-10-10-52.png)

#### Database tables

Now that we've covered the application itself, we'll take a quick tour of the two database tables used by the application. In this overview I'm using SQL Server Management Studio to view the database tables.

##### Firewall log entries table
First we see the table of firewall log entries which are used in the application. This persistent storage approach allows the application to still use the historical data in the event it is restarted. The careful reader will notice the Geolocation entries are not normalized; this is due to the fact that IP addresses change location over time. However, I am exploring the possibility of adding at least one more table to further normalize the database.

[![](/assets/2022-11-21-firewall-status-display/2022-11-25-10-06-34.png)](/assets/2022-11-21-firewall-status-display/2022-11-25-10-06-34.png)

##### Geolocation table
This is where we find mappings of IP address ranges to geolocation data. I chose to use this in lieu of a cloud-based web API due to rate limits and other considerations. However, in the future I do plan on implementing a database-less approach to this.

[![](/assets/2022-11-21-firewall-status-display/2022-11-25-10-24-58.png)](/assets/2022-11-21-firewall-status-display/2022-11-25-10-24-58.png)


## Evaluation

In this section we'll evaluate the effectiveness of this solution using three criteria:
- Analysis of the firewall data itself
- Use with different router firmware & Ubuntu Linux
- Requirements and setup comparison to other solutions

### Pt. 1 - Firewall data

We'll examine various aspects of the data in more detail and attempt to draw some conclusions about what happened. With this data set, we are working with approximately 39 days of data ranging from 10/11/22 to 11/24/22. There are around 246,000 log entries averaging 6,304 dropped packets per day.


#### Traffic by country
Using the geolocation information in the database, we're able to see where this traffic originates from. 

[![](/assets/2022-11-21-firewall-status-display/2022-11-24-11-19-00.png)](/assets/2022-11-21-firewall-status-display/2022-11-24-11-19-00.png)

Here is the table corresponding to the top 10 origin countries:
|Country|Packets|Percentage of traffic|
|-------|-------|---------------------|
|USA            |73,234 |28.54%|
|Netherlands    |51,853 |20.21%|
|Russia         |27,203 |10.6%|
|China          |16,464 |6.42%|
|Cyprus         |12,556 |4.89%|
|Canada         |11,170 |4.35%|
|UK             |10,074 |3.93%|
|Germany        |8,805 |3.43%|
|South Korea    |3,895 |1.52%|
|France         |3,074 |1.2%|

As we can see from the table we have the usual suspects of the USA, China, and Russia. Surprisingly, the Netherlands came #2 on this list. At this time it's unclear why, however, I suspect part of the reason is due to the large industry of PaaS providers based in this region. In my own experience of procuring Virtual Private Server (VPS) slices and cloud-based servers I discovered a large amount of them are based in the Netherlands and surrounding areas.

The value of this data is being able to mitigate swaths of attacks by blocking all traffic originating from a given country. There are several scripts online which can do this job, they typically involve a series of iptables rules which filter traffic by geographic region, again using publicly available geolocation tables.

#### Traffic by port

The application provides a similar breakdown of traffic by port, which we will review here.

[![](/assets/2022-11-21-firewall-status-display/2022-11-25-12-52-20.png)](/assets/2022-11-21-firewall-status-display/2022-11-25-12-52-20.png)

Here is the table of top 5 ports:
|Service Name|Port|Packets|Percentage|
|------------|----|-------|----------|
|Telnet|23|23280|9.16%|
|SSH|22|2750|1.08%|
|Http|8080|2746|1.08%|
|Personal-agent|5555|2418|0.95%|
|SIP|5060|1435|0.56%|

As we can see, telnet continues to be a very common attack vector well into late 2022. This is likely due to the large amount of misconfigured systems still remaining in the wild, such as routers, servers, and any other device which utilizes a control channel using the telnet protocol. Such devices will either have a blank password or a default password (with the device name sometimes conveniently placed in the logon banner).

Also, SSH and http (using port 8080, an alternate HTTP port) seem to be common attack vectors. The SSH attempts are most likely to be probes to identify servers for bruteforcing attempts, while the http port 8080 connections are likely to log into routers using the remote administration web interface. Such routers can be left unprotected if the user doesn't correctly configure them, allowing an adversary to gain control of the router through this port.

Notably absent are traffic on http/https (ports 80 and 443, respectively). This is due to the fact that COX cable filters tcp port 80. I also have my own web service running on port 443 for https connections, so these do not show up as dropped packets. It's entirely likely that a large amount of reconnaissance/attacks were attempted on these ports but will not show up in this analysis.

### Pt. 2 - Different firmwares

We'll now review how well the application performed on 3 platforms. I will give an overview of my experience and explain what I encountered with each one. Overall, the application proved itself to be robust in terms of differing platforms.

#### OpenWRT

This was the native development platform and thus all contingencies for this were built into the application during the development of the packet parser.

Setup was straightforward, OpenWRT has the firewall enabled by default with a relatively strict filtering policy. I needed only to enable firewall logging using the web interface, and then enable remote syslog using SSH to change a configuration file. Packets logged in such a fashion have an input interface name of "wan" and a ""DROP" tag.

[![](/assets/2022-11-21-firewall-status-display/2022-11-25-16-37-14.png)](/assets/2022-11-21-firewall-status-display/2022-11-25-16-37-14.png)


#### DD-WRT

DD-WRT logs the firewall packets by default and needed only to be configured to output syslog entries to a remote system. The fields of the firewall data were the same, however, I did come across an issue where the actual remote syslog packets were truncated to 256 bytes. This can be overcome with further configuration and/or a switch to another remote syslog package, but as it still provided all information relevant for my analysis, I didn't pursue this.

Like OpenWRT, all packets from the DD-WRT router have a "DROP" tag. It can be differentiated from OpenWRT by the interface name of "vlan2".

[![](/assets/2022-11-21-firewall-status-display/2022-11-25-16-38-01.png)](/assets/2022-11-21-firewall-status-display/2022-11-25-16-38-01.png)


#### Ubuntu Linux
I used Ubuntu Linux 22.04 for this demonstration. As I was not able to procure purpose-designed hardware in time, I set up a Virtual Machine on my main server which has a 4-port NIC card. I was then able to set up a WAN/LAN bridge to emulate an edge Linux router for the purposes of this evaluation. The VM ran on Microsoft Windows 2019 w/Hyper-V.

Setup of the Ubuntu Linux firewall logging was similar to setting it up with DD-WRT. I enabled the firewall using the ufw command and it immediately logged to the local syslog. After that, I set up remote logging using a configuration file and it worked pretty well. The only tweak I had to do was add a regular expression to check for the tag, which was "[UFW BLOCK]". The interface name was "eth0" as I had assigned the first virtual NIC to the WAN port.

[![](/assets/2022-11-21-firewall-status-display/2022-11-25-16-40-38.png)](/assets/2022-11-21-firewall-status-display/2022-11-25-16-40-38.png)


### Pt. 3 - Vs different solutions
I'm going to evaluate requirements & setup against other solutions. Here they are:
- Graylog
- Splunk
- Alienvault OSSIM

## Limitations

Limitations of the solution are mostly obvious. They arise from the deadline to project completion, the scope of the project itself (this program won't make your coffee), and it's purpose to fulfill a niche need. We'll go over some of the specific limitations now.

### SQL server express size limitations

SQL server express itself is a significant limiting factor for the application. With a 10GB database limit, one can expect approximately 15 to 20 days of historical data. For the purposes of my evaluation I used SQL Server Developer Edition instead, which has no limit. So SQL Server Developer Edition is a possible way around this limitation (until MySQL support is added in a future release), depending on the use case.

### Minimal ad-hoc reporting

This limitation is due to the time constraint. The application produces a few reports such as traffic by country/port and port scans, however, there is no generalized reporting function. Future iterations may have this capability, and even sooner, an ability to export firewall data to CSV format for use in applications such as Microsoft Excel.


### No cross-platform support

The program is only verified to work in Windows 10, although I highly suspect it will also work in Windows 11 with the same pre-requisites met. Because it is a Windows Presentation Foundation (WPF) application, much re-work will be needed to make it truly cross-platform supported. However, there is the possibility that this app will still run in a Windows API framework such as Wine, though I haven't tested this hypothesis.


### Scope is limited to firewall data only (by design)

This program is designed to fulfill a niche and to do one task and aims to do it well. It displays firewall data and attempts to give insights and alert the user of potential historical and ongoing attacks. That's all it does. It is not meant to be a comprehensive security solution but rather to work in tandem with other approaches, or at the very least, serve as a stop-gap measure until the user can implement a broader-scope security solution down the road. The program is therefore designed with this scope and these constraints in mind.

In addition, the system is not designed to act as an IDS or an IPS (such as IPFire), though it may seem so at first. Keep in mind that currently, the firewall program only displays *dropped* packets, which indicates such attacks were thwarted.  Future iterations of this solution will likely include support for allowed packets for a more comprehensive look into active sessions.

Finally, one can only view the data in the program and it therefore offers no remediation for an ongoing attack. Such information alerts the user only, and it will then be incumbent upon them to take the appropriate action to mitigate the threat. It is expected that the target demographic of this application will have the skills necessary to stop an attack, or at the very least, the willingness to unplug their network cable in an emergency.

## Future work

We'll now discuss future work, which I believe there is a lot of potential for. Not only have there been many good suggestions from my colleagues, but even many of my own ideas were not implemented simply due to the time constraint. This program still has potential for further development and this version is simply a base example of what could be done in the time allowed.

### Support for multiple RDBMS

Currently the program supports Microsoft SQL Server, which can be free in certain cases (SQL express w/10GB database size limit, SQL Server Developer Edition), but may not be appropriate for all users. Thus there is a need to expand database support for relational database alternatives such as MySQL.

### Further UI integration & enhancements

The UI has basic functionality but could some basic enhancements:
- Copying data from the tables
- Exporting to CSV, PDF or other formats
- Enhanced UI integration such as context menus, right clicking for focused views, etc
- Graphs need cleaning up

### Database-less option

The original intended design did not have a database table for the Geolocation entries; this was added to mitigate the problem of rate limits from free Geolocation web APIs. Working with these rate limited APIs would have involved additional complexity in the program such as queues and timers. Such an approach is still feasible, however, I de-prioritized it there was limited time to develop the application. That said, there is still a good use-case for a database-less option for this application, as the entire purpose is to avoid having to install heavy back-end infrastructure.

The limitations of such an approach are possible delays in retrieving Geolocation data and only having a non-persistent store of firewall log data, but these tradeoffs are likely acceptable to many users. 

### More dashboards & alerts


#### Add filtering capability to dashboards
#### Add further customization for types of alerts
### Hints for IPTables rules based on pertinent traffic
### Also log allowed & rejected traffic 
## Conclusion
### So why bother using this?
#### There are other potential good solutions to tackle this problem but this fulfills a niche
#### Further investment in hardware not usually required if already able to run OpenWrt, DD-WRT, or Linux as a router
#### Runs on Windows 10 with over 75% market share
#### Alerts appear in the system tray almost in real-time
#### High functionality out-of-the-box, minimal customization
#### Relatively low prerequisites and requirements
### Solution is ideal for intermediate to advanced users who want a working solution quickly without much tinkering
## References
Vailshery, L. S. (2022, November 22). IOT connected devices worldwide 2019-2030. Statista. Retrieved November 22, 2022, from https://www.statista.com/statistics/1183457/iot-connected-devices-worldwide/ 

Zou, X. (2022, August 18). IOT devices are hard to patch: Here's why-and how to deal with security. TechBeacon. Retrieved November 22, 2022, from https://techbeacon.com/security/iot-devices-are-hard-patch-heres-why-how-deal-security 

Desktop windows version market share worldwide. StatCounter Global Stats. (2022, October). Retrieved November 23, 2022, from https://gs.statcounter.com/os-version-market-share/windows/desktop/worldwide 

## Appendix