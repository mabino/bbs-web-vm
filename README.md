# Self-Hosted Bulletin Board System (BBS) Configuration

These directions aim to create a terminal experience for a visitor to a self-hosted, residential Bulletin Board System (BBS) without requiring anything more than a modern web browser.

Visitors load a secure web page hosted on one of two virtual machines (VMs) running [Ubuntu 20.04 LTS](https://ubuntu.com) atop a [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/) host.  A combination of [Apache Web Server](https://httpd.apache.org), [websockify](https://github.com/novnc/websockify), [fTelnet](https://www.ftelnet.ca), and an [SSH](https://www.openssh.com) tunnel on the first VM wrap the telnet session to an installation of [Synchronet](http://www.synchro.net) BBS via secure [WebSocket](https://datatracker.ietf.org/doc/html/rfc6455) (wss) on the second VM.

These directions assume a number of prerequisite skills, capabilities, and available resources to achieve this goal, specifically but not limited to the creation and configuration of virtual machines with Hyper-V, an understanding of network routing and addressing, a solid, foundational knowledge of both Windows and Linux operating systems, as well as posession of hardware with two physical, wired network adapters and, ideally, two separate local area network (LAN) segments with their own, independent wide area network (WAN, e.g. Internet) access.

## Web Server Virtual Machine (web.vm)

## Bulletin Board System Virtual Machine (bbs.vm)
