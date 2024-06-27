# Networks-admin

![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/560eb884-e669-44df-ad5f-85fa5b8a42e6)


 I usually troubleshoot layer 1 and layer 2 in union because they are so closely paired. Examples of layer 2 – data link – are line protocol (such as Ethernet, ATM, 802.11, PPP, frame-relay, HDLC, or PPP).


To troubleshoot at these layers, the first thing I would do on your router is a show interface. Here is an example of a LAN Gigabit Ethernet circuit:

![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/fafe33ce-156c-4e0f-83a1-042d71732a5f)


Here is what a WAN T1 or T3 circuit might look like:

![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/13c78c41-fce1-4e2f-ae94-566bcd5878ee)


Here is the quick version:


![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/a05173a1-c1e1-4f6b-ac8e-0e32e3a7ec0e)



We look for the following:

    Is the interface UP?
    Is the line protocol UP?
    If both the interface and line protocol are NOT up, your connection is never going to work.
    To resolve a line down, I look at the cable or the keepalives
    To resolve a line protocol down, check to make sure that the protocols match on each side of the connection(notice the “line protocol” on each of the interfaces above).
    Are you taking input, CRC, framing, or other errors on the line (notice how the serial interface above does show errors)? If so, check your cable or contact your provider.

In general, I verify that we have a good cable on each side, verify that line protocols match, and that clocking settings are correct.



If this is an Ethernet connection, is there a link light on the switch?

If this is a serial connection, do you have an external CSU/DSU? If it is an external CSU, check that the Carrier Detect (CD) light & data terminal ready (DTR) lights are on. If not, contact your provider. This also applies if you have an internal Cisco WIC CSU card. If that is the case, take a look at this Cisco link on understanding the lights on that card.

You can, of course, use the Cisco IOS test commands to test your network interfaces with internal staff and with your telecommunications providers.

Do not proceed to upper level layers until your Physical interface on the router shows as being UP and your line protocol is UP. Until then, don’t worry about IP addressing, pinging, access-lists or anything like that.


![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/b345a07c-490b-4c3c-b461-374a65f5178a)



Once we have Layers 1 & 2 working (your show interface command shows the line is “UP & UP”, it is time to move on to layer 3 – the OSI Network layer. The easiest thing to do here to see if layer 3 is working is to ping the remote side of the LAN or WAN link from this router. Make sure to ping as close as possible to the router you are trying to communication with – from one side across to the other side.

Here are examples of successful & failed pings:


![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/fa6d222b-f4c0-47e2-9160-176f2fa294b7)


The easiest way to check the status of Layer 3 – the network layer – is to do a show ip interface brief, as I did above. Here is an example:

![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/6e602d4d-794d-47ef-901f-64cac4f5a768)


Notice the IP addressing on each of these interface. Also do a show running-config, like this (you can even specify an interface, like this):


![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/448baacb-f382-4812-83d3-a482917e372d)



 would recommend taking this interface configuration and comparing it, side by side, with the remote WAN connection to ensure they are the same. Ask yourself questions like:

    Are these interfaces on the same IP network?
    Do these interfaces have the same subnet mask?
    Are there any access-lists (ACL) that are blocking your traffic?
    Can you remove all optional IP features to make sure that the basic configuration works before adding additional features that could be causing trouble?

Here is an example. Look at the two interfaces below. What is the real problem, causing these two to not communicate?

Router 1

interface Serial3/0 description Sprint T3 – TO ROUTER 2 bandwidth 9000 ip address 10.2.100.2 255.255.255.252

Router 2

interface Serial3/0 description Sprint T3 – TO ROUTER 1 bandwidth 1500 ip address 10.2.100.5 255.255.255.252

No, there is no problem with the bandwidth statement. Bandwidth statements are only used as comments and by routing protocols to select the best route. The real problem here is that the second router’s serial interface is not on the same IP subnet as router #1. Even though they have the same subnet, the 10.2.100.5 IP address will never be able to communicate to the 10.2.100.2 IP address because they are on different networks but directly connected.

Let’s say that you are now able to ping across the link, from one side to another. While that is a great sign, it doesn’t always mean that everything is “fixed”. You still may not be able to communicate from a client on the LAN of one router, to a client on the LAN of another router, due to things like improperly configured IP routing protocols.

For one LAN to communicate to another LAN, through routers (through a WAN, usually), you MUST have either static routes  or dynamic routes configured. To ensure you have a route configured for the network you are trying to reach, do:

Router# show ip routes

and look at

Router# show ip protocols

For troubleshooting layers 3, all the way up, look at the output of this command:


![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/a5e58da6-09b2-48fa-a9ed-27248f1e58b7)


![image](https://github.com/ammaramin763/Networks-admin-skills/assets/162980561/eca116a9-a700-4c93-9307-3ab03834ff9f)



Now, let’s say that you have made it to the point where you can ping from LAN to LAN, through your WAN. Congratulations – that is a very good sign. If you are still having trouble, it must be in OSI Layers4-7. Here are those layers listed out and possible issues you might experience in each layer:

    Layer 4 – Transport – in the transport layer are TCP and UDP – you could be have an ACL or QoS feature blocking or slowing this traffic. Your TCP traffic could also be fragmented to the point that it could not be reassembled. Another option is that you may not be receiving an ACK back from your traffic that was successfully sent.
    
    Layer 5 – Session – in the session layer are protocols like SQL, NFS, SMB, or RPC – you could be taking errors on any one of these session protocols. I would recommend using a protocol analyzer like Wireshark to analyze your session data.
    
    Layer 6 – Presentation – in the Presentation layer are data encryption, compression, and formatting – your VPN tunnel could be failing or perhaps you are sending one type of data (like a MPEG) and the receiver is trying to view it as a WMV file.
    
    Layer 7 – Application – in the Application layer are, of course, your applications like FTP, HTTP, SCP, TFTP, TELNET, SSH, and more – you could be trying to connect to a telnet server with the SSH protocol, for example.
    
    Layer 8 – End User – the standing joke is that “Layer 8” is the user – the user could be just mistyping their username or password or you, the network admin, could have been troubleshooting the wrong IP address all along.



















Troubleshooting OSPF on a Cisco Router



![image](https://github.com/ammaramin763/Networks-admin/assets/162980561/27f7d80f-e0ea-4598-bbdb-be730718a39e)


We can begin our troubleshooting by looking at the output of the ‘show ip ospf neighbor’ command.

![image](https://github.com/ammaramin763/Networks-admin/assets/162980561/5dca4de4-d014-4551-9cbc-814e46a8eb9a)

Here Neighbor id is important since this shows the linked neighbor or Router id.


