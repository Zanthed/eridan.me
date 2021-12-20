---
title: "(WIP) Sysadmins Guide to Java Minecraft Server Hosting"
date: 2021-11-10T00:00:00-00:00
draft: true
summary: " "
---

This article is currently a WIP and things will change at any point in time. Due to the amount of time this is taking to write and research, I have committed it and will update it as I go on.

> Note! If you came here for a "how to optimise my minecraft server" guide, you can leave and find a different article. This article is not how you can optimise it, but rather how you can deploy a server properly, scale it well, and maintain it all while being highly performant and low latency. This is to also teach you inner workings of Java Minecraft to better understand what configuration you should do.

> This article tries to be somewhat agnostic or generalised, but I may include software-specific things such as configuration files for certain Minecraft server software, operating systems, etc.
> This article also does not intend to be a "what VPS do I buy". I put down things you should take into consideration and contact a salesperson to discuss about, or set up the infrastructure to. I will not vouch for any provider.

Off that note, a lot of people who want to make a Minecraft server don't actually know just how weird it is. It's not hard at all. It is not hard to buy a cheap 60 dollar Raspberry Pi kit off of Amazon, flash Raspbian OS on that poor basic SD card, and use a random frontend script someone made from GitHub. Not hard.

Problem is when people want to do a little more than play with their friends, or run some more intensive things, they greatly underestimate or don't realise how resource intensive and complicated it is to make the experience enjoyable. Even something like Redstone requires a bit more power than a Raspberry Pi.

"But my server runs fine!" Yeah, packets don't travel that far if your server is on the same network as you are, or how little processing 4 players in the same chunk do. Or this article simply does not apply to you.

{{< table_of_contents >}}

## Planning
You need to figure out who your target audience is. Are you running a simple Vanilla survival server for your friends? Or is it a public survival server that intends to be for everyone to play. Any mods? If so, Fabric or Forge? Or any plugins will be used? What is your target demographic location? Is it east coast? Or west coast? Or your entire country? Are they in the same state as you? Do you have IPv4 only? IPv6 only? Or both!

Is this a simple LAN server? A public server? Is there a whitelist or not?

(Note: This list of questions are arbitrarily picked out and you need to figure out what questions, listed here or not, are relevant to you.)
These questions are all pertinent that require answers to determine things like your security model, optimise networking for LAN or WAN, and general configuration to make the server accessible.

For example, you may want to make a public survival server for not just your friends, but they may be the most active players on 80% of the time. Your friends are also in the same state as you. Self hosting may be an option with a higher security model in mind.

## Hardware and Software
### Bandwidth and Network
Support IPv6. Please. It is absolutely not considered experimental and we are recycling IPv4 addresses which has shown adverse<sup>[i]</sup> affects<sup>[i]</sup> to servers. IPv6 supports IPSec and IPSec Encapsulating Security Payload (ESP), even when behind NATs. Modern versions of Minecraft support IPv6 just fine. (Laugh at the people who refuse to implement IPv6 or applaud anyone who has added support for IPv6.)

Depending on who your target audience is, high bandwidth may not be necessary, but it is always desirable no matter the circumstances to have low latency. Latency is a far worse thing to play against than speed. Minecraft doesn't need a lot of bandwidth for a few players, but I personally suggest anything at least 200Mbps (upload).

To combat latency, strictly avoid copper-heavy infrastructures. Don't use incredibly long cables. Standard Ethernet cables are copper, but the travel distance is very minimal compared to entire ISP infrastructures that are copper. Get fibre, no questions asked. Make sure it's Fibre to the Home or Building. Fibre to the Neighborhood/Node is still fibre, however you're routed with copper to that fibre node which helps with download speeds, but upload speeds and latency will suffer badly still. Charter Communications is a common offender of this and cap you at best 65Mbps upload even when paying for their gigabit plan, which is an awfully low amount of upload speed.

Routers are a real pain. Networks are not trustworthy at all, or secure in any way, and replacing your old router with garbage, bloated, and insecure firmwares like DD-WRT/OpenWrt or OPNsense does not improve security at all. You'd want a HardenedBSD router with aarch64 and even then networks are still by design fundamentally insecure and clients should distrust it at all costs. Routers fail to implement security features like verified/measured boot, disk encryption, security kernel modules, rollback protection, common exploit mitigations, or have any form of hardware security module (HSM) to make any of this even useful or possible. While you can build your own router and firewall and theoretically implement all of this yourself, it's highly non-trivial and people severely underestimate the amount of work this would take.

From a performance standpoint, routers still fail horribly. Unless you use Broadcom chipsets with the proprietary blobs or Intel chipsets, very few other chipsets match performance similar to those two. Open source implementations of Broadcom exist, but they're very lackluster in features like enterprise WPA2 and incredibly slow in performance due to lack of hardware acceleration. Intel has developed official open source drivers. Realtek is a particularly buggy and inconsistent chipset from both the hardware and software side. FreeBSD barely supports it, and even then your speeds are really flakey.

In the Server Operating Systems and Hypervisors section, I talk about NIC bonding/teaming with schedulers like 802.3ad (link aggregation) and round-robin.

The next few sections will talk about software threading and synchrony.

### How VPS's work
VPS's are shared hosting. It should be obvious you are not going to get a dedicated server with purely physical hardware just for you unless you purchase a dedicated server specifically. That's expensive for both the provider and you. To make room for everyone, providers give you shared resources through virtualisation and a (usually type 1) hypervisor; also usually Linux KVM and scripting QEMU or another KVM frontend, and UEFI being TianoCore EDK II. You have multiple people on one processor and even multiple people on the same physical core as you. Server load can determine how much power is available to you dynamically. If the servers are overloaded, your performance will suffer. If servers are not overloaded, you have more headroom.

### Software Threading
Software executes code usually using both multi-threaded and single-threaded mechanisms. Threading is all about workers. Who is doing what? The term "thread" here is not how many cores your processor has. A thread is the stream of execution and execution context. Single-threading uses just one of those. It's how all functions are executed by default; call a function, does a thing, returns a thing. Simple.

Multi-threaded software execute code using multiple execution contexts, or threads. You do a thing, do another thing, and another thing all at the same time. In some cases, this can be a bit risky, what is known as not thread-safe, if you are doing this to do more work like trying to render things and the drawing process was made to draw in a sequential manner.

### Synchronous v. Asynchronous
Now in functions, you obviously do stuff in them. Since threading is a stream of execution, a program executes stuff in a sequential manner by default. This is where synchronous and asynchronous come in. Contrary to popular teaching: synchronous, asynchronous, single-threaded, and multi-threaded are NOT at all the same. Synchrony and asynchrony are all about tasks.

Synchronous executes stuff in a specific sequence and a function is supposed to finish before doing the next thing. This is not the same as single-threaded because you can execute synchronous things with multiple threads. A common example is before HTTP/2, it was a synchronous request/reply system.

Asynchronous, on the other hand, let's you do a thing and it does not block the flow of execution. You can execute asynchronous tasks under a single thread. HTTP/2 is asynchronous as it uses multiplexing and any resources that failed to load do not block loading the website.

The real difficulty is when you do asynchronous multi-threading. You now have multiple workers doing multiple tasks all at the same time. You need to implement measures to make sure they don't interfere or access a resource already in use (mutex locking and unlocking), deadlocks, and race conditions. But if all done correctly, an intensive task can be done significantly faster.

See these great Stackoverflow<sup>[i]</sup> answers on threading<sup>[i]</sup> and synchrony.

### Relation to Java Minecraft
Before getting to the VPS part, let's wrap it up by explaining how this ties in with Java Minecraft.

You're probably familiar with the infamous never-ending losing war of how terribly unoptimised chunk rendering and chunk generation is in Minecraft. There are plenty more examples, but these two have been problems since day-one. Minecraft heavily relies on single-threaded synchronous operations, being on the main thread. Even as of 1.17, Minecraft is still extremely single-threaded reliant even on chunk operations. There are some multi-threaded or asynchronous operations like disk I/O and Netty supporting multiple threads. But even then, async and multi-threading are not magic performance changers or black or white if there are many<sup>[i]</sup> other<sup>[i]</sup> problems<sup>[i]</sup>. Notch admits Java not being the fastest.

It is so difficult to implement asynchronous execution on many things on Java Minecraft, that even PaperMC<sup>[i]</sup> developers<sup>[i]</sup> have made jokes about it or repeat themselves because it's not magic, black or white, or trivial<sup>[i]</sup> to implement<sup>[i]</sup>. Asynchronous tasks for Minecraft will forever be experimental, no matter how much people push for it and promote new "revelations in async technology". You need to focus on solving other problems with your setup instead of telling yourself async is a magic bullet. Plugins need to be async and thread-safe and thousands of lines of code in Minecraft need to be rewritten to be async and thread safe.

### Relation to VPS's
So, it's safe to say you need high single-core performance to compensate for the poor programming, and you definitely don't want a shared vCPU. Providers may offer what is known as "dedicated CPU usage" or just a dedicated CPU. You still have a virtual machine given to you, but you are given a dedicated CPU core(s) that are not shared. These are going to be your best bet for high core performance, even on multi-threaded operations for things like your operating system. No one else is on these cores, but you. For Minecraft, it's suggested at least 2 or 4 dedicated cores. They may cost a tad bit, but not as crazy as buying a full dedicated server.

### "Need to buy more RAM!"
You're not wrong. Java Minecraft is extremely needy of memory, no matter what it is you are doing. Aikar, a PaperMC developer, stresses the importance of using 6-10GB of memory *minimum*, no matter how many players you have or what you're doing to give plenty of head room for the GC. Unfortunately, RAM has turned into a marketing term for Minecraft server hosting. Providers love stressing about how they have so much memory and charge at either too cheap prices or too high prices, and all still have poor quality because they miss more key things as I have mentioned before relating to processing power and load. Stay away from Minecraft server hosting providers, but you will definitely need a lot of memory here. Make sure you leave headroom memory for the operating system too, something people easily forget.

Error correction code memory (ECC) DIMMs are hardware memory modules (DIMMs) that attempt to correct any memory errors that may occur, such as cosmic bit flipping or other non-deterministic sources of memory interference, especially on the cells. ECC can mitigate row hammer bitflips on the DIMMs as well. Because of how memory intensive Minecraft is and to avoid unexplainable weird errors that may be caused by it, try to consider it when possible as it's not<sup>[i]</sup> totally<sup>[i]</sup> far<sup>[i]</sup> fetched<sup>[i]</sup>.

### Storage
Just buy NVMe. Please. Though SATA RAID 10 can provide improved performance and data integrity compared to typical SATA if you cannot get NVMe. Make sure you do not purchase DRAMless SSDs. They are not useless, but in a server environment their purpose is out of their league here. Especially as you start to fill up the SSD and the performance boggles down to even slower than 7200 RPM HDDs because lack of DRAM cache.

### Multi-Socket Motherboards and NUMA
Multi-socket, typically dual-socket, motherboards are not black/white. They don't magically "team" together to do processing. Non-uniform memory access (NUMA) is used to achieve this. NUMA requires a fair bit of configuration and may be a learning curve for some people. Software needs to be NUMA aware (Java supports this with the -XX:+UseNUMA flag) and thread affinity is necessary for resource management and delegation to make the fullest extent of performance possible with multiple processors and keep latency low (some software can handle this itself by making it NUMA aware). If you do happen to understand NUMA very well and know how to set it all up correctly, it would definitely yield great results only on newer Java versions.

## Server Operating Systems and Hypervisors
If you purchased a full dedicated server or have the hardware for, I suggest looking into Windows Server and Hyper-V with Arch Linux as the guest. Hyper-V features core scheduling<sup>[i]</sup> for SMT to decide on improved performance or stricter guest isolation, Generation 2<sup>[i]</sup> VMs support larger drives (GPT), safer disk images with VHDX (and higher performance!)<sup>[i]</sup>, safer and easier migration and backups, and less legacy code or hardware used<sup>[i]</sup>. SR-IOV compatible network adapters are natively supported by Hyper-V<sup>[i]</sup>.

Avoid Virtualbox at all costs. It has very weak guest isolation due to the bloat/absurd amount of guest-host features<sup>[i]</sup>, a terrible security<sup>[i]</sup> track record<sup>[i]</sup>, poor security<sup>[i]</sup> researcher communication<sup>[i]</sup>, and is only a type-2 hypervisor.

If you don't have a dedicated server, consider Arch Linux as your OS. Avoid package freezing or "stable" distro release models due to issues such as downstream kernels, package freezing, failure to keep up with security patches on both their software and the kernel, unnecessary downstream patching<sup>[i]</sup>, lots of unnecessary post-install scripts<sup>[i]</sup> that are prone to breakage and actually confuse the user, having to deviate from using CVEs and use your own vulnerability ID system<sup>[i]</sup>, and many more. Debian, Manjaro, Ubuntu, and RHEL are well-known offenders here, Debian being the absolute worst with security vulnerabilities still unpatched and are being actively exploited in the wild. Even Debian admits their Chromium package is highly insecure and outdated.

Rolling release is not precisely that "unstable". A lot of instability is from trying to do too many things with one single host, or from over-using the AUR or other "non-typical" ways of obtaining software like maintaining a bunch of tarballs and keeping up to date and compatibility. Downstream package-freezing distros are a lot more prone to breakage than rolling because of downstream patching and developers maintaining new downstream codebases. When errors occur, upstream solutions don't work or they're not even an upstream issue in the first place.

For Arch Linux and general Linux security advice and a hardening guide, refer to madaidan's Linux Hardening guide. (Linux is not a secure operating system.)

Linux may be generally preferred though as the kernel natively supports NIC teaming/bonding and configuration for 802.3ad (link aggregation) to improve network stability and throughput or round-robin for load balancing. To get the fullest improvements of using 802.3ad, you need hardware (switch and NICs) compatible with it, otherwise your LACP Rate may be set to slow instead of full. A slow LACP Rate means one LACPDU (LACP Data Unit, basically a LACP packet) is sent every 30 seconds, where as fast is one LACPDU every second. This doesn't mean your speeds are slower, it's just the effectiveness of link aggregation here isn't as boosted compared to normal. systemd-networkd provides easy configuration frontend to use this. Windows does not natively support bonding and requires usage of vendor software like Intel ANS teaming.

systemd, despite being massively bloated and has a lot of attack surface, supports granular service sandboxing through unit files. This can assist in isolating and sandboxing your server, which you should do. Solutions like<sup>[i]</sup> firejail<sup>[i]</sup> are<sup>[i]</sup> quite<sup>[i]</sup> questionable<sup>[i]</sup>. bubblewrap is the best so far, but it's not too user-friendly and needs manual configuration. It's also essentially necessary to combine it with syscall filtering using seccomp rather than just bubblewrap only and mandatory access control (MAC) like AppArmor or SELinux, but both also require manual configuration and setting up a strict MAC is extremely tedious due to desktop/server software being written with an outdated 90s security model.
