# Self-Hosted Bulletin Board System (BBS) Configuration

These directions aim to create a terminal experience for a visitor to a residentially self-hosted Bulletin Board System (BBS) without requiring anything more than a modern web browser.

Visitors load a secure web page hosted on one of two virtual machines (VM) running [Ubuntu 20.04 LTS](https://ubuntu.com) atop a [Hyper-V](https://docs.microsoft.com/en-us/virtualization/hyper-v-on-windows/) host.  A combination of Apache Web Server, [websockify](https://github.com/novnc/websockify), [fTelnet](https://www.ftelnet.ca), and an SSH tunnel wrap the telnet connection to an installation of [Synchronet](http://www.synchro.net) BBS via a secure websocket (wss) connection.
