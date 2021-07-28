# pihole-blocklist
The session-reply and host file were taken from https://switchedtolinux.com/privacy-resources/

The hosts file is the same as stlblock.txt but is using 127.0.0.1 instead of 0.0.0.0. The 0.0.0.0 is for pi-hole and to apply to the entire LAN.

Steps to my local build Pihole + WireGuard + Unbound
1. Install Pihole on RaspberryOS
5. Initial settings don't matter since we'll override them.
6. Install list tool: https://github.com/jessedp/pihole5-list-tool
7. Install auto-update lists from remote sources: https://github.com/jacklul/pihole-updatelists
8. Install whitelist: https://github.com/anudeepND/whitelist
9. Install PiVPN with Wireguard.
10. Use Install Unbound without DoH/DoT. 
11. Install https://my.zerotier.com/network/a09acf0233f9a6bf as an alternative to WireGuard (instructions: https://discourse.pi-hole.net/t/how-to-easily-use-your-pi-hole-outside-of-your-personal-network/18878)


Steps to my cloud build Pihole + WireGuard + Unbound

1. Sign-up for Oracle Cloud Infrastructure Free Tier
2. Choose the free services
3. Select Ubuntu 20.04
4. Validate public IP will be available
5. Generate SSH keys and paste them before creating the image
6. Once image created, when viewing your instance in the control panel, under “Primary VNIC” click “Public Subnet”
    Then click “Default Security List for vnc-0000000”
    Click “Add Ingress Rules”

Rule 1:

    Source Type: CIDR
    Souce CIDR: 0.0.0.0/0
    IP Protocol: UDP
    Destination Port: 51820
    Description: WireGuard UDP
    Click the blue “Add Ingress Rules” button.

Back on the main page of the instance:
7. scroll down and click “VNIC” on the left in the side bar under Resources and edit your VNIC.
8. Check “Skip source/destination check”
9. Remove IPtables restrictions

$ sudo iptables -L 

Then I saved the rules to a file so I could add the relevant ones back later:

$ sudo iptables-save > ~/iptables-rules 

Then I ran these rules to effectively disable iptables

by allowing all traffic through:

$ iptables -P INPUT ACCEPT $ iptables -P OUTPUT ACCEPT $ iptables -P FORWARD ACCEPT $ iptables -F 

To clear all iptables rules at once, run this command:

$ iptables --flush 

10. Install Pihole and do not enable DNSSEC

        IF YOU WISH TO CHANGE THE UPSTREAM SERVERS, CHANGE THEM IN:          
                      /etc/pihole/setupVars.conf                             
                                                                             
        ANY OTHER CHANGES SHOULD BE MADE IN A SEPARATE CONFIG FILE           
                    WITHIN /etc/dnsmasq.d/yourname.conf

11. Create a file in /etc/dnsmasq.d/yourname.conf
12. Inside the file type: server=127.0.0.1#5335
13. Once done, install PiVPN $ curl -L https://install.pivpn.io | bash and follow the instructions

Disable resolvconf for unbound (optional)¶

**UNBOUND**
The unbound package can come with a systemd service called unbound-resolvconf.service and default enabled. It instructs resolvconf to write unbound's own DNS service at nameserver 127.0.0.1 , but without the 5335 port, into the file /etc/resolv.conf. That /etc/resolv.conf file is used by local services/processes to determine DNS servers configured. If you configured /etc/dhcpcd.conf with a static domain_name_servers= line, these DNS server(s) will be ignored/overruled by this service.

To check if this service is enabled for your distribution, run below one and take note of the the Active line. It will show either active or inactive or it might not even be installed resulting in a could not be found message:

`sudo systemctl status unbound-resolvconf.service`

To disable the service if so desire, run below two:

`sudo systemctl disable unbound-resolvconf.service`

`sudo systemctl stop unbound-resolvconf.service`

To have the domain_name_servers= in the file /etc/dhcpcd.conf activated/propagate, run below one:

`sudo systemctl restart dhcpcd`

And check with below one if IP(s) on the nameserver line(s) reflects the ones in the /etc/dhcpcd.conf file:

`cat /etc/resolv.conf`


Creating A DNS Only Tunnel / Split-Tunnel in WireGuard

Obviously with WireGuard we can create a full tunnel, but with this setup we likely just want to reduce this to DNS traffic.

How do we do this? Limit AllowedIPs in your wireguard client configuration.

Example client configuration:

[Interface]
PrivateKey = <YOUR PRIVATE KEY>
Address = 10.6.0.3/24 # This is the assignable range for this client
DNS = 10.6.0.1 # This is the IP of pihole.[Peer]
PublicKey = <YOUR PUBLIC KEY>
PresharedKey = <YOUR PRESHARED KEY>
AllowedIPs = 10.6.0.1/32 # Important part 0 Set it to DNS IP above.
Endpoint = <YOUR PIHOLE PUBLIC IP>:51820 

What does this do? By setting AllowedIPs to be equal to the pihole IP, it will only allow traffic that is directed at our DNS (i.e. pihole) to be sent through the tunnel. Effectively creating a split-tunnel.

    
