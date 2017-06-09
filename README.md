# Useful Crypto
Some useful commands and patterns for authentication, verification, and key management.

## Self-Hosted VPN on AWS
Performance of free AWS tier is good enough for web browsing. Throughput is good (25 MB down/2 MB up) but latency is poor (~250ms-700ms).
### AWSLinux Micro w/ OpenVPN (free)
- [Manual Setup Guide on Comparitech](https://www.comparitech.com/blog/vpn-privacy/how-to-make-your-own-free-vpn-using-amazon-web-services/).
- Uses single preshared private key.

### RHEL 7.x Micro w/ OpenVPN (free)
Setup was a pain. May you have more luck.
See (COMING SOON): `[rhel7.x.openvpn.yaml or .sh](rhel7.x.openvpn.yaml)`
- Uses tls, certificate authority (ca), easy-rsa
- To run: `rhel7.x.openvpn.yaml [port]`

### Load-Balancing w/ MultiVPN
See (COMING SOON): Launch N free micro proxies, automatically load balance TCP connection between them.
- Experimenting with HAProxy and the native OpenVPN functionality. Would be nice to mix and match between TCP proxy variants.

## Else-Hosted VPN
- [ExpressVPN](https://www.expressvpn.com/)
- [IPVanish VPN](https://www.ipvanish.com/)

## Password Manager: [drduh/pwd.sh](https://github.com/drduh/pwd.sh)
- A very simple gpg-based password manager.

## SSH and Port Forwarding
### SSH-in-SSH
`$ ssh -L2008:host2:22 host3`
In new shell:
`$ ssh -p 2008 localhost`
or:
```
host host2
  hostname localhost
  port 2008
```
`$ ssh -L2008:host2:22 host3`
`$ ssh host2`

### Generating an SSH Key
#### 1. Generate Keypair w/ Random Identifier:
`$ ssh-keygen -t rsa -b 4096 -C $(openssl rand -hex 20)`
#### 2. Specify Destination:
`~/sshdir/keyname.key`
#### 3. Generate Random Password:
`$ openssl rand -hex 20`
#### 4. Correct File Permissions
`$ chmod 700 ~/sshdir/keyname.key.pub`
`$ chmod 600 ~/sshdir/keyname.key`
#### 5. Add IdentityFile under ~/sshdir/config (eg. Github):
```
Host github.com
    HostName github.com
    IdentityFile ~/sshdir/github.key
```
#### 6. Load key to external site and test:
`$ ssh -T git@github.com`

## Key Operations
### Load Specific Key:
`$ ssh-add ~/sshdir/github.key`
### Load All Keys:
`$ ssh-add ~/sshdir/*.key`
### List Keys:
`$ ssh-add -l`
### Unload Keys - all or specified:
`$ ssh-add -D || -d <key>`

## MAC Randomization
- `sudo ifconfig en0 ether $(openssl rand -hex 6 | sed 's/\(..\)/\1:/g; s/.$//')`

Recommend [SpoofMAC](https://github.com/feross/SpoofMAC) to randomize MAC address whenever needed, eg:
- Randomize for all devices on system startup.
- Randomize on change of network location.

## MAC Hijacking for Free Wifi - Bypass MAC-based Authentication
This one is so easy you can't not do it:
1. Connect to open Wifi network. You will get a DHCP address (eg. 192.168.1.100) and the router stores your current MAC as 'unauthenticated'.
2. Check for other hosts on network:
```
$ arp -a
? (192.168.1.1) at 88:15:44:ac:a1:dd on en0 ifscope [ethernet]
? (192.168.1.89) at 1:0:5e:0:0:fb on en0 ifscope [ethernet]
? (192.168.1.13) at 1:0:5e:7f:ff:fa on en0 ifscope [ethernet]
```
3. Note the gateway router and any potential targets:
```
Router: (192.168.1.1) at 88:15:44:ac:a1:dd
Host1: (192.168.1.89) at 1:0:5e:0:0:fb
Host2: (192.168.1.13) at 1:0:5e:7f:ff:fa
```
4. Open Wifi routers usually maintains a table of authenticated users tracked based only on their MAC addresses (or MAC + IP Address). Only these users are permitted to traverse the NAT and access the internet.
5. Thus, assigning an already authenticated MAC gives you internet access. Try host1: `sudo ifconfig en0 ether 1:0:5e:0:0:fb`, test for internet, then try host2 if needed.
6. You may also need to assign a new IP address and subnet mask to make it work.
