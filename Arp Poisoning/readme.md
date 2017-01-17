This is based off of a Hack UCF talk in the Fall semester of 2016

#What is ARP?
To start off what is Address Resolution Protocol? It is the protocol that maps physical address (MAC Address) to a network address (IP address). Why do we need this? Let’s say you are playing League of Legends. Your computer sends data to the server using the server’s public IP address. Thing is it doesn’t just go straight to the server, there are a lot of stops before it gets there (like your switch, your router, your isp’s router). Each one of these moves is known as a hop. Each time your data is transferred over a hop, it moves using a physical address, not on ip address. That is why we need ARP.

#How does ARP work?
When a computer joins a computer, it sends out ARP requests, to which the router and some other computers will reply for with their mac address. It will go ahead and store those in a table, which it will use when it needs to send data. Keep in mind this is only for computers in your own subnet or vlan (your piece of the network). You can view your ARP data in Windows or Linux by typing the following command.

```
arp -a
```

Here is an example of an arp table from Windows 8.1, which has been compromised by ARP Poisoning.

```
Interface: 172.16.213.128 --- 0x3
  Internet Address      Physical Address      Type
  172.16.213.1          00-50-56-c0-00-08     dynamic   
  172.16.213.2          00-0c-29-76-7c-9b     dynamic   
  172.16.213.134        00-0c-29-76-7c-9b     dynamic   
  172.16.213.254        00-50-56-ee-e5-e7     dynamic   
  172.16.213.255        ff-ff-ff-ff-ff-ff     static    
  224.0.0.22            01-00-5e-00-00-16     static    
  224.0.0.251           01-00-5e-00-00-fb     static    
  224.0.0.252           01-00-5e-00-00-fc     static    
  239.255.255.250       01-00-5e-7f-ff-fa     static    
  255.255.255.255       ff-ff-ff-ff-ff-ff     static    
```

Here is an example of an arp table from ubuntu.

```
? (172.16.213.134) at 00:0c:29:76:7c:9b [ether] on vmnet8
router.asus.com (192.168.1.1) at 30:85:a9:68:70:00 [ether] on wlp7s0
? (172.16.213.128) at 00:0c:29:4f:99:18 [ether] on vmnet8
```

As you can see with both examples, they both store the ip address and the associated MAC Address. 

#ARP Poisoning
What ARP Poisoning is, is when we manipulate ARP Requests and Responses so that a target machine will store a bad MAC address for an IP Address. For instance ARP Poisoning could change the MAC Address for a computer’s default gateway to that of our computer, so it would send all of it’s traffic to a computer of our choosing. This would allow the attacker to monitor all traffic, modify the traffic, block it, and other things. Keep in mind that the computer that we send the traffic to has to be on the same network segment (on the same switch). You can refresh the table by rebooting.

#How do I do ARP Poisoning?
For Linux, we are going to use a tool called Ettercap (Ettercap comes default on Kali). For Windows you can use Cain and Abel. We will review how to do it in terminal, then with the GUI. For the CLI * first fire up a terminal then run the following command to scan for all hosts that are on your subnet. This will attack all computers on the subnet

```
Ettercap -T -M arp:remote
```

The “-M” flag indicates a man in the middle attack. The “arp:remote” designates that it will be an arp spoofing attack using the remote option, so that it will forward the packets back to your machine, and then to default router so it will still work.

The “-T” flag will indicate that you want to run it in text mode, so you will get to see a lot of the output.

Also let’s say that you want to only attack one target. Let’s say that the target is 172.16.132.128. You would add the following option to the command.

```
//172.16.132.128//
```
Now here is what the arp table looks like after we launch this attack. You can tell that it has been compromised since there are multiple IP addresses with the same Physical Address. This should never be the case under normal circumstances.

```
Interface: 172.16.213.128 --- 0x3
  Internet Address      Physical Address      Type
  172.16.213.1          00-0c-29-76-7c-9b     dynamic   
  172.16.213.2          00-0c-29-76-7c-9b     dynamic   
  172.16.213.134        00-0c-29-76-7c-9b     dynamic   
  172.16.213.254        00-0c-29-76-7c-9b     dynamic   
  172.16.213.255        ff-ff-ff-ff-ff-ff     static    
  224.0.0.22            01-00-5e-00-00-16     static    
  224.0.0.251           01-00-5e-00-00-fb     static    
  224.0.0.252           01-00-5e-00-00-fc     static    
  239.255.255.250       01-00-5e-7f-ff-fa     static    
  255.255.255.255       ff-ff-ff-ff-ff-ff     static   
```

Now for the graphical environment. First click on sniff on the taskbar, then star a Unified sniffing (Unified will sniff all packets on the cable, whereas bridged will forward traffic from one interface to another and sniff all of the packets). You need to choose your primary ethernet interface (probably eth0). After that click on Hosts in the taskbar, then scan for hosts. After that it should populate a list of hosts. Right click on the default gateway, and press “Add to Target 1”. Then click on the victim and press “Add to Target 2”. After that click on “Mitm” (Man In The Middle) and then select “Arp Poisoning”. You will be given the option between remote, and one way. Essentially the difference is remote will poison the gateway so it will send response data your way to sniff, and one way won’t (whcih will be helpful since it will avoid potential scanning software on the switch). After that you can fire up Wireshark or another packet sniffing utility and do whatever you want.

#How to stop ARP
Arp Poisoning is a pretty common exploit, so there is a large array of security implementations for defending against this.

1.) Firewall - If a Firewall is implemented properly (default block incoming), it will block certain fake ARP requests. Keep in mind that this can still be worked around, so it shouldn’t be your only solution.

2.) Manually Check - This is one of the least effective methods. You could check every so often, then if there are any you could just clear your arp cache. In Windows the command is “netsh interface ip delete arpcache” and for Ubuntu it is “ip -s -s neigh flush all”

3.) Static ARP - You could set static ARP entries. You will essentially set all of the entries static. To statically add ARP entries in Windows 8.1, you can use the following command

```
Netsh -c interface ipv4 add $adapter $ip $MAC
```

or to do it in Ubuntu 

```
sudo arp -s $ip $MAC”
```

Note: With VM’s you may run into an issue where the VM isn’t running in promiscuous mode, and 
because of that it can’t do some of the activities. To rectify that, run the following command on your host machine however ensure that the network option is changed to whatever virtual network you are using.
Source:
http://vmware.com/info?id=161











