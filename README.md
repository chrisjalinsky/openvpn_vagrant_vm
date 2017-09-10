# PiVPN Vagrant Box Testing

All credit goes to this awesome project:
[http://www.pivpn.io](http://www.pivpn.io/)

## Vagrant and Ansible

This guide assumes that Vagrant, Ansible, and Virtualbox are present.

This Ansible and Vagrant setup creates a bridged VM OpenVPN server on your local network, for testing purposes.

**NOTE:** This guide assumes DHCP exists on the network to provide the VM with an IP address. If not, the bridged Vagrant Virtualbox setup will not work properly without handling this another way.

All this project truly does is provides the "simulated" hardware needed to run this server.
The installation method is meant for a Debian system, and Vagrant/Virtualbox provides this, Ubuntu 14/16.

**Bridge**
open the Vagrantfile and set the correct "BRIDGE_IFACE" variable:
I'm testing on a Macbook, so it's bridge is en0.

To run:
```
vagrant up
vagrant ssh
```
Once inside the machine:
```
vagrant@vpn1: $ sudo su -
root@vpn1: # cd /tmp/
root@vpn1: # ./install_vpn.sh
```

### Notes about the install dialog on the vagrant machine:
* The bridge is located on the eth1 interface, use this during the installation dialog
* Use the vagrant user
* Use your networks DNS server to resolve other machines on your network, or Google: 8.8.8.8, 8.8.4.4
* Use TCP on a higher port, Certain ISPs will block udp and port 1194. for instance tcp on 44347
* For testing purposes, use the 1024 bit encryption algorithm...it's much faster

### Notes about firewall/router
* You'll need to port forward from Firewall/router to this VM
* For instance, if you configured the OpenVPN to serve tcp on port 44347, on your firewall, port forward inbound tcp 44347 to the VMs tcp port 44347

To verify service on the VMs interface and port:
```
root@vpn1:~# netstat -ntple

Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       User       Inode       PID/Program name
tcp        0      0 0.0.0.0:111             0.0.0.0:*               LISTEN      0          7906        639/rpcbind
tcp        0      0 0.0.0.0:50866           0.0.0.0:*               LISTEN      107        7977        676/rpc.statd
tcp        0      0 0.0.0.0:22              0.0.0.0:*               LISTEN      0          9002        1093/sshd
tcp        0      0 0.0.0.0:44347           0.0.0.0:*               LISTEN      0          9090        1132/openvpn
```
You can see that the openvpn service is running on all interfaces, and port 44347.

To verify ip address to add port forwarding rule to firewall. You can see the eth1 interface has received a local lan ip address, and this ```192.168.10.54``` is the address to port forward to.
```
root@vpn1:~# ifconfig
eth0      Link encap:Ethernet  HWaddr 08:00:27:4d:bb:01
          inet addr:10.0.2.15  Bcast:10.0.2.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2152 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2007 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:144194 (144.1 KB)  TX bytes:197312 (197.3 KB)

eth1      Link encap:Ethernet  HWaddr 08:00:27:d3:42:fe
          inet addr:192.168.10.54  Bcast:192.168.10.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:2385 errors:0 dropped:2 overruns:0 frame:0
          TX packets:22 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:169922 (169.9 KB)  TX bytes:3552 (3.5 KB)
```
