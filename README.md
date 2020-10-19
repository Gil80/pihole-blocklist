# pihole-blocklist
The session-reply and host file were taken from https://switchedtolinux.com/privacy-resources/

The hosts file is the same as stlblock.txt but is using 127.0.0.1 instead of 0.0.0.0. The 0.0.0.0 is for pi-hole and to apply to the entire LAN.

Steps to my build:
1. Install Pihole on RaspberryOS
3. Enable DNSSEC
4. Install https://developers.cloudflare.com/1.1.1.1/dns-over-https
5. Use 1.1.1.2 and 1.0.0.2
6. Install list tool: https://github.com/jessedp/pihole5-list-tool
7. Install auto-update lists from remote sources: https://github.com/jacklul/pihole-updatelists
8. Install whitelist: https://github.com/anudeepND/whitelist
9. Install PiVPN with Wireguard
10. Use Install Unbound? that might be instead of DNS over HTTPS (DoH).
11. Install https://my.zerotier.com/network/a09acf0233f9a6bf
