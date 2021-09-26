# Self-Hosted Bulletin Board System (BBS) Configuration

These directions aim to create a terminal experience for a visitor to a self-hosted, residential Bulletin Board System (BBS) without requiring anything more than a modern web browser.

Visitors load a secure web page hosted on one of two virtual machines (VMs) running [Ubuntu 20.04 LTS](https://ubuntu.com) atop a [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/) host.  A combination of [Apache Web Server](https://httpd.apache.org), [websockify](https://github.com/novnc/websockify), [fTelnet](https://www.ftelnet.ca), and an [SSH](https://www.openssh.com) tunnel on the first VM wrap the telnet session to an installation of [Synchronet](http://www.synchro.net) BBS via secure [WebSocket](https://datatracker.ietf.org/doc/html/rfc6455) (wss) on the second VM.

The directions assume a number of prerequisite skills, capabilities, and available resources to achieve this goal, specifically but not limited to the creation and configuration of virtual machines with Hyper-V, an understanding of network routing and addressing, a solid, foundational knowledge of both Windows and Linux operating systems, as well as posession of hardware with two physical, wired network adapters and, ideally, two separate local area network (LAN) segments with their own, independent wide area network (WAN, e.g. Internet) access.

The architectural design and software selection prioritize simplicity of deployment and automation of updates, while the BBS customization prioritizes the availablity of Door games and basic bulletin board functionality and presentation.  The principles of this project:

* Run an early 90s era BBS software and host it via a residential ISP
* Eliminate speciality hardware and software requirements for visitors to the extent possible
* Provide games, BBS artwork, and menuing
* Create a sustainable and secure backend with a modern-ish stack

## Web Server and Bulletin Board System (BBS) Virtual Machine Shared Steps

1. Configure both the Web Server and the BBS virtual machines to prioritze the correct routing for your setup.  For instance, in this example configuration of the file `/etc/netplan/00-installer-config.yaml`, both network devices receive routing information via DHCP.  However, because `eth0` is the public-facing device designed to serve most of the incoming web and WebSockets requests, it is important that packets get routed back to that connection first.  Routing to the internal network first may result in timeouts, particularly on a residentially-hosted connection.

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

### GoDNS

If hosted via a dynamically assigned IP address that you wish to map to a domain name, download and compile [GoDNS](https://github.com/TimothyYe/godns) and move the binary into `/usr/local/bin`.  Move [configs/godns.conf](configs/godns.json) into `/usr/local/etc` and use [godns.service](https://github.com/TimothyYe/godns/blob/master/systemd/godns.service) in `/etc/systemd/system` for GoDNS service startup.  This service can exist on the web server or any other server that reports outbound traffic via the public IP.

`systemctl enable godns.service ; systemctl start godns.service`

The GoDNS service works with multiple, well-known domain name registrars.  In this example `/usr/local/etc/godns.json` file, the registrar is Dreamhost and authentication is via an API login token obtained from the account.  The arrays with keys `domain_name` and `sub_domains` catch all CNAME entries via `@`, so enumerating the individual sub-domains is unnecessary provided they all point to the same address.

```
{
  "provider": "Dreamhost",
  "login_token": "<your token>",
  "domains": [{
      "domain_name": "<your domain>",
      "sub_domains": ["@", "<sub domain 1>", "<sub domain 2>"]
    }
  ],
  "resolver": "8.8.8.8",
  "ip_url": "https://myip.biturl.top",
  "ip_type": "IPv4",
  "interval": 300,
  "resolver": "ns1.dreamhost.com",
  "socks5_proxy": ""
}
```

## Bulletin Board System Virtual Machine (bbs.vm) Specific Steps

1. Expose the Nested Virtualization Extensions for Hyper-V.

`Set-VMProcessor <VMName> -ExposeVirtualizationExtensions $true`

2. Install dosemu, add `/usr/lib/dosemu` to `/etc/ld.so.conf.d/libc.conf`, and run `sudo ldconfig`.

## Supplemental TradeWars Game Server

1. Create a user that will run the graphical desktop environment for the Trade Wars Game Server.

```
sudo adduser --disabled-password twgs
```

2. Install `x11vnc`, `xvfb`, and `fluxbox`.

```
sudo apt-get install x11vnc xvfb fluxbox
```

3. Set an x11vnc password for use by the `twgs` user.

```
sudo su - twgs
x11vnc -storepasswd yourVNCpasswordHERE /home/twgs/x11vnc.pass
```

4. Create a startup script for `xinit` and `fluxbox` at `/etc/systemd/system/xinit.service`.

```
[Unit]
Description="xinit as twgs"

[Service]
User=twgs
Group=twgs
Type=simple
ExecStart=/usr/bin/xinit /usr/bin/fluxbox -- /usr/bin/Xvfb :20 -nolisten tcp
ExecStop=/usr/bin/killall xinit
#Restart=on-failure

[Install]
WantedBy=multi-user.target
```

5. Create a startup script for `x11vnc` at `/etc/systemd/system/x11vnc.service`.

```
[Unit]
Description="x11vnc as twgs"
Requires=xinit.service
After=xinit.servic

[Service]
Type=simple
ExecStart=/usr/bin/x11vnc -env DISPLAY=:20 -rfbauth /home/twgs/x11vnc.pass -forever
ExecStop=/usr/bin/killall x11vnc
Restart=on-failure

[Install]
WantedBy=multi-user.target
```

6. Create a startup script for the Trade Wars Game Server software at `/etc/systemd/system/twgs.service`.

```
[Unit]
Description="twgs"
Requires=x11vnc.service
After=x11vnc.servic

[Service]
User=twgs
Group=twgs
Type=simple
Environment="DISPLAY=:20"
ExecStart=/usr/bin/wine '/home/twgs/.wine/drive_c/Program Files (x86)/EIS/TWGS/twgs.exe'
ExecStop=/usr/bin/killall wine
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
