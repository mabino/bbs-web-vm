# Self-Hosted Bulletin Board System (BBS) Configuration

These directions aim to create a terminal experience for a visitor to a self-hosted, residential Bulletin Board System (BBS) without requiring anything more than a modern web browser.

Visitors load a secure web page hosted on one of two virtual machines (VMs) running [Ubuntu 20.04 LTS](https://ubuntu.com) atop a [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/) host.  A combination of [Apache Web Server](https://httpd.apache.org), [websockify](https://github.com/novnc/websockify), [fTelnet](https://www.ftelnet.ca), and an [SSH](https://www.openssh.com) tunnel on the first VM wrap the telnet session to an installation of [Synchronet](http://www.synchro.net) BBS via secure [WebSocket](https://datatracker.ietf.org/doc/html/rfc6455) (wss) on the second VM.

The directions assume a number of prerequisite skills, capabilities, and available resources to achieve this goal, specifically but not limited to the creation and configuration of virtual machines with Hyper-V, an understanding of network routing and addressing, a solid, foundational knowledge of both Windows and Linux operating systems, as well as posession of hardware with two physical, wired network adapters and, ideally, two separate local area network (LAN) segments with their own, independent wide area network (WAN, e.g. Internet) access.

The architectural design and software selection prioritize simplicity of deployment and automation of updates, while the BBS customization prioritizes the availablity of Door games and basic bulletin board functionality and presentation.  The principles of this project:

* Run an early 90s era BBS software and host it via a residential ISP
* Eliminate speciality hardware and software requirements for visitors to the extent possible
* Provide games, BBS artwork, and menuing
* Create a sustainable and secure backend with a modern-ish stack

## Web Server and Bulletin Board Virtual Machine Shared Steps

```
network:
  ethernets:
    eth0:
      dhcp4: true
      dhcp4-overrides:
        route-metric: 50
    eth1:
      dhcp4: true
      dhcp4-overrides:
        route-metric: 100
  version: 2
  ```

## Web Server Virtual Machine (web.vm) Specific Steps



## Bulletin Board System Virtual Machine (bbs.vm) Specific Steps

## Supplemental TradeWars Game Server

1. Install `x11vnc`, `xvfb`, and `fluxbox`.

```
sudo apt-get install x11vnc xvfb fluxbox
```

2. Set an x11vnc password.

```
sudo x11vnc -storepasswd yourVNCpasswordHERE /etc/x11vnc.pass
```

1. Create a startup script for `x11vnc` at `/etc/systemd/system/x11vnc.service`.

```
[Unit]
Description="x11vnc"
Requires=display-manager.service
After=display-manager.service

[Service]
ExecStart=x11vnc -create -env FD_PROG=/usr/bin/fluxbox -env DISPLAY=:0 -env X11VNC_CREATE_GEOM=1024x768x16 -bg -rfbauth /etc/x11vnc.pass -forever
ExecStop=/usr/bin/killall x11vnc
Restart=on-failure
Restart-sec=2

[Install]
WantedBy=multi-user.target
```
