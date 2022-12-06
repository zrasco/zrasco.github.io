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

During the evaluation I set an upper bound time limit of 3 hours of configuration effort per product. While I'm convinced all of these could eventually work, 3 hours seems sufficient to form a basis of comparison to my own solution (which takes around a half hour to set up total).

#### Graylog

I have an existing graylog instance in my environment which I use for other tasks. When doing my initial research it seemed as though Graylog could support firewall data in a similar fashion so I decided to see if it were possible.

The initial setup and configuration of Graylog was done in the past and took approximately 8 hours. At first I did install the Graylog downloadable virtual appliance, but then decided to install the server onto an existing Linux VM. This VM has 2 CPU cores, 8GB of RAM and approximately 700GB of SSD storage (I use this as a file server as well), 200GB of which is dedicated to Graylog logs.

Unfortunately, I ran into similar issues as with Alienvault OSSIM (please see next section). While I was able to set up the firewall as an event source, forward packet data to Graylog, and show the raw data on the screen, this was as far as I could go. While there does appear to be support for certain types of vendor firewalls in Graylog via third party dashboards (such as Palo Alto) coupled with Graylog's paid Enterprise product, Graylog does not seem to have a parser for breaking iptables firewall data up into discrete fields.

In conclusion I was not able to find the correct set of criteria needed to successfully integrate iptables logs into Graylog in any fashion beyond simply displaying the raw log output in a dashboard. While this may be possible, the time needed to set up Graylog (over 8 hours) was already well above the time needed to set up my own solution and thus we have evaluated Graylog according to the scope of the evaluation framework.   

[![](/assets/2022-11-21-firewall-status-display/2022-12-04-19-14-21.png)](/assets/2022-11-21-firewall-status-display/2022-12-04-19-14-21.png)


#### Splunk

I opted for the 1.4GB Splunk OVA (virtual appliance) to attempt a quick evaluation, which required me to create an account. This is essentially a virtual machine image which is designed to quickly drop into an existing VMWare environment. It's typically pre-configured and runs out of the box without much setup needed.

Immediately I was faced with an issue, the Splunk OVA required 8 CPUs but my ESXI host only has 4. Despite changing the VM settings to 4 CPU cores, I continued to receive errors regarding a lack of CPU resources and was ultimately unable to get it working. The only host I have with 8 CPU cores is my Hyper-V server, which unfortunately rules out the possibility of using an OVA appliance (there are ways to convert from OVA to a Hyper-V hard disk but it seems an install from scratch is more sensible as there may be unforeseen consequences of the OVA conversion).

I was ultimately able to install Splunk on a Hyper-V VM.

Setup of the VM and Splunk itself was relatively simple. After this, I redirected the router syslog to UDP port 514 of the splunk VM and attempted to install the IPTables Netfilter visualization app, and it's dependencies, the TA_netfilter app and the Splunk CIM app, into Splunk to show a visualization of the firewall data.Unfortunately, the app did not show the initial setup screen when ran.

I uninstalled the app and made another attempt installing both the IPTables app and TA_netfilter using the GUI tool. After about 45 minutes, I was able to get a dashboard up with some basic firewall information such as packet volume over time, port numbers, and so forth. However, IP and Geolocation information was still absent so I worked to configure these as well. I was able to determine the logical splunk fields were not being parsed and populated correctly (such as clientip, country, city, and so forth) so I worked on this. The issue appears to be rooted in the TA_netfilter dependency.

After much trial-and-error I was able to determine that TA_netfilter is not outputting fields into the new event source that are required by the IPtables Netfilter Splunk app. I spent time reviewing the documentation for both apps and still was not able to retrieve the relevant fields needed to show geolocation and IP address information as needed by the dashboards. At this point I have hit the 3 hour mark and have at least partial success, so I consider this a sufficient attempt for the purposes of this evaluation. I do believe it's possible to get the Geolocation and IP address information correctly in the dashboards, but was not able to determine how to do so in the 3 hour window.

[![](/assets/2022-11-21-firewall-status-display/2022-12-06-12-08-46.png)](/assets/2022-11-21-firewall-status-display/2022-12-06-12-08-46.png)


#### Alienvault OSSIM

To set up AlienVault OSSIM 5.8.11, I provisioned a VM in line with the recommended requirements: 2 CPU cores, 8GB of RAM and 128GB of SSD space. I installed it on an ESXI host using a Debian 8 64-bit profile.

During installation, it hung during the step displaying "Configuring alienvault-gvm11-feed (amd64)". I was finally able to go online and figure out a way to make the installation proceed, but had I left it alone it may have ended there. Total installation time to first boot logon screen took about 26 minutes.

Initial configuration took around 6 minutes and was straightforward. I had to configure the network interfaces and discovered devices. I determined that my router was not in the list of discovered devices and attempted to add it as an event source.

I reconfigured my router to send UDP syslog packets to the AlienVault OSSIM VM and used the AlienVault OSSIM troubleshooting tools (netstat and syslog) to confirm the packets were being received by AlienVault's VM. However, they would not show up in the web interface. I spent about 2 more hours of research to figure out a way to incorporate these packets into the UI, but was unsuccessful.

Additionally, I looked into the various dashboards available and could find none that showed the Geolocation or port information for firewall packets. Built-in dashboards only show generic information such as authentication, authorization, and so forth. While this additional insight may ultimately be somehow available, I was not able to find it during the evaluation period.

I'm therefore forced to conclude that while AlienVault OSSIM may be a viable all-purpose solution for many types of threats, it does not have specific firewall insight capabilities out of the box. I suspect that doing so would require a large degree of customization, possibly into the realm of setting up an unsupported configuration. However, I am not an expert in AlienVault OSSIM so it's possible I might have missed something.

Time spent for this evaluation: Approximately 3 hours. Result: Inconclusive, but likely not supported.

#### IPFire

I will briefly mention IPFire because it contains similar features and can even act as an IPS. However, because of the hardware requirements, IPFire is not a direct competitor in terms of this evaluation. That said, I fully endorse IPFire as an alternative solution to my project, provided the reader has adequate physical equipment. 

### Pt.3 - Vs different solutions (summary)

Of all 3 solutions, the only one I was able to get partially working in the time alloted was Splunk. It also seemed possible to do so with Graylog, but unlike Splunk, there was no out-of-the-box log parser available for syslog-style iptables log entries. Such a parser may be available or can be created, but I was unable to find it, and obviously creating one would go well beyond the evaluation constraints. As for AlienVault OSSIM either does not support this type of reporting environment at all, or would do so only after heavy customization which would include custom log ingestion, creation of data adapters, creation of visualization dashboards, custom geolocation integration, and so forth.

In addition, the infrastructure requirements for all 3 comparable solutions involve provisioning a separate VM of varying configurations, which require more time (sometimes setting up an OS) and which requires more infrastructure than my project, which will install on a Windows 10 machine in a matter of minutes and work basically out-of-the-box. Thus based on this evaluation I conclude and state that my application fulfills the needs/goals set out in the design, specifically, to provide a feature-rich solution with minimal effort compared to other solutions.

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

### Further evaluation against other comparable products
Because this project fulfills such a niche, it has proven difficult to find an apples-to-apples product to evaluate this against. I have made a best-effort attempt to do so, however, my lack of finding an exact product to compare to does not necessarily indicate such a product doesn't exist. Further research is possible to find similar products which may result in a more detailed comparison.

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

The dashboards that are presently in the application were put in during the time allowed to show the basic functionality and are not as flexible as they could be. For example, filters could be added to the dashboards to allow a finer-grained look into the data. In addition, the graphs could be further customized to allow different axes of data or time periods to be displayed. Finally, there is no general-purpose ad-hoc report module, which would allow various types of graphs and reports to be created, templates to be saved, and so on. Therefore there is much development potential in this area of the application.

The same can be said with alerts. While it was my intention to include more than 1 type of alert (currently port scans), time ran short and thus port scan alerts are the only type available. Furthermore, this type of alert is minimally customizable (it can be tweaked in appsettings.json). Further types of alerts can be added via an alert management module which abstracts the types of data flows triggering such alerts. Parameters such as types of packets, frequency of packets, size, etc... can all be input as parameters into this module and custom alerts can be created which the user determines would require some type of immediate action.

### Hints for IPTables rules based on pertinent traffic

While the program acts as a view-only, the fact that it uses iptables logs also potentially provides the capability of outputting iptables commands which will further mitigate detected threats. For example, with the geolocation database there is the potential to automatically generate a script consistenting of iptables commands which will block entire countries. Another possible scenario is if patterns of port scans are detected, to discard all packets from these IP addresses and any associated Autonomous Systems (AS's). The iptables command-line interface is nearly universal across all platforms that support it and thus the program has the potential to suggest helpful commands to the user to strengthen their security posture.

### Also log allowed & rejected traffic 

The implementation presented in this current body of work logs dropped traffic only. This is for several reasons:
- Dropped traffic is typically already logged with minimal customization
- Logging allowed/rejected traffic substantially increases overhead and customization requirements
- Such logging is implementation-specific

Each flavor of the custom firmware, as well as Ubuntu Linux, have optimal mechanisms for logging this type of traffic and each requires specific customization. However, despite the increased difficulty, the main benefit is that this sort of logging can alert the user to a focused attack. Examples include bruteforce attempts, SYN floods and other similar types of DoS attacks, and many more. The user can then take appropriate action to mitigate the attack. Combined with the suggested iptables hints rules mentioned above, this may give even an inexperienced user the ability to stop an ongoing attack with relative ease.

## Conclusion

We'll wrap things up with a summary of why this program is useful, the advantages/disadvantages of using it, and which needs it fulfills.

Yes, there are other products that can fulfill this task that are out there. However, there are none (as of the time of this writing) that fulfill this specific niche/task. This program is designed to do one thing and do it well... report firewall activity for a large body of existing, inexpensive infrastructure that is already out there. With minimal setup requirements and configuration compared to other solutions, this application satisfies the need for a user to hit the ground running quickly with minimal hassle. It reports firewall data and does not seek to do anything else.

Further hardware investment is not typically required, provided the user has a router that can receive custom firmware such as DD-WRT. VM infrastructure does not need to be provisioned for virtual appliances. The program is open-source and can be adapted by any competent person willing to add features. It runs on Windows 10 which has a high market share of the desktop market, which also allows desktop integration features such as notifications/alerts. There is minimal configuration required, and the main features of the program work out-of-the-box. The system requirements are modest, requiring only SQL Server (with support for other relational databases and/or a database-less option later), the .NET 6 runtime, and a mere 150MB of disk space.

In summary, I argue this program fits a need for the target audience of intermediate to advanced users who want to monitor their firewall activity inexpensively, quickly, and effectively with minimal headaches. It ultimately aims to fulfill the niche it's intended to serve and I believe this work is a promising start and has strong future potential.

## References
Vailshery, L. S. (2022, November 22). IOT connected devices worldwide 2019-2030. Statista. Retrieved November 22, 2022, from https://www.statista.com/statistics/1183457/iot-connected-devices-worldwide/ 

Zou, X. (2022, August 18). IOT devices are hard to patch: Here's why-and how to deal with security. TechBeacon. Retrieved November 22, 2022, from https://techbeacon.com/security/iot-devices-are-hard-patch-heres-why-how-deal-security 

Desktop windows version market share worldwide. StatCounter Global Stats. (2022, October). Retrieved November 23, 2022, from https://gs.statcounter.com/os-version-market-share/windows/desktop/worldwide 

## Source code snippets used

## Instructions/sources used for evaluation

Alienvault installation instructions: https://cybersecurity.att.com/documentation/usm-appliance/initial-setup/ossim-installation.htm

Graylog installation instructions & links to virtual appliances: https://docs.graylog.org/docs/installing

Spunk OVA for VMWare: https://splunkbase.splunk.com/app/3216

Installation of Splunk on Ubuntu Linux: https://www.bitsioinc.com/install-splunk-ubuntu/

IPtables for Splunk: https://readthedocs.org/projects/iptables-for-splunk/downloads/pdf/latest/


## Appendix