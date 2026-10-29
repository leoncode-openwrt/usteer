# LeonCode Usteer maintenance

`main` preserves two independently reviewable changes on the OpenWrt
history:

- `91a7369`: bound peer updates to MTU-safe UDP datagrams.
- `9479697`: initialize the socket before sending the first update.

The matching message-size and IPv4/IPv6 socket regression tests are retained.
Original author information remains intact. Source changes are pinned by full
commit, not selected from a moving branch during a release build.

[LeonCode packages](https://github.com/leoncode-openwrt/packages) builds the
ARM64 and MIPS package variants with each supported OpenWrt release SDK.
Publishing a package never installs it on a device or changes steering settings.
