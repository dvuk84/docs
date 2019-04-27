This guide will explain how to set up a Checkpoint VPN client on Ubuntu.

**1. install tools needed for l2tp/ipsec vpn to work**

```bash
sudo add-apt-repository ppa:nm-l2tp/network-manager-l2tp  
sudo apt update  
sudo apt install network-manager-l2tp network-manager-l2tp-gnome ike-scan
```

**2. scan the vpn server for config info**

```bash
sudo ike-scan <VPN_IP>
```

You should see something like this:

```
Starting ike-scan 1.9.4 with 1 hosts (http://www.nta-monitor.com/tools/ike-scan/)  
<VPN_IP> Main Mode Handshake returned HDR=(CKY-R=d3d5531f7a714d60) SA=(Enc=3DES Hash=SHA1 Auth=PSK Group=2:modp1024 LifeType=Seconds LifeDuration(4)=0x00007080) VID=f4ed19e0c114eb516faaac0ee37daf2807b4381f000000010000138d5ba4ce9a0000000018380000 (Firewall-1 NGX)  
Ending ike-scan 1.9.4: 1 hosts scanned in 0.028 seconds (35.67 hosts/sec).  1 returned handshake; 0 returned notify`
```

Note the encryption and hash type above. You will need that later.

**3. create new VPN connection**

Under your Connections you should now see an option to add an IPsec/L2TP type VPN. Create a new connection.

- add a _Gateway_ <VPN_IP>
- add your username and password
- click _IPsec Settings_ and enable _IPsec tunnel to L2TP host_
- add a pre-shared key
- under _Advanced_ add encryption and hash that you got from ike-scan result
 - _Phase1_: 3des-sha1-modp1024
 - _Phase2_: 3des-sha1

- in _PPP settings_ untick all authentication types except _PAP_ and set the MTU to 1500

**4. add default route**

Create a new file in _/etc/ppp/ip-up.d/defaultroute_ and make it executable.

```bash
#!/bin/sh  
ROUTE=$(ip r | grep default | awk '{print $3}')  
route add <VPN_IP> gw $ROUTE
```

Now you should be able to connect to the VPN from the Network menu.
