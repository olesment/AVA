## Http on TCP on IP
TCP session allows the program to send stream of bytes back and forth over the network
TCP is in the middle layer of protocol stack 
HTTP and other applications are built on top pf TCP
TCP is built on top of IP [_internet protocol_]
Upper levels of protocols depend on behavior of lower levels while hiding lower level details, giving programmers a simpler interface

## The protocol stack

Web applications rely on HTTP to send requests about resources and get responses
Flask or requests are ue to build web applications
Wbapps make use of fetures browser and server already implement
> protocol: Http
> 
> > Concepts: resources, verbs, cookies\
> > The code is in Flask, Apache, browsers\
> > failures: error codes, slow responses. Any misbehavior of lower level is visible at HTTP layer\

> TCP\
> > Concepts: Ports, sessions, stream sockets\
> > Where the code is: OS kernel, system library\
> > broken connections, timeouts
> > Reliable connection between device and server is provided by OS TCP implementation\
> > TCP relies on Lower level protocol - the IP - to deliver packets between hosts and pass them through routers\
> > Failures: broken connections timeouts\

> IP\
> >Concepts: IP addresses, packets\
> > The code is: in OS kernels, routers\
> >  Failures: various\

>WiFi
>> Concepts: Access points, WPA passwords\
>> Where the code is: in device drivers\
>> failures: network unavailable\

Not all network aplication use TCP, PING uses ICMP
DNS uses UDP.
NTP [_network time protocol_] keeps time and date in sync

> Application
> >Http, SSH,  NTP

>Transport
>> TCP, UDP

> Internet
> > IP

> Hardware
> > Ethernet, WiFi

When talking about this make sure you say on what layer you are, because a lot of terms and concepts are shared between layers.

## Ping and DNS in tcpdump
tcpdump `tcpdump`shows whats ping actually doing [linux]\
TCPDUMP(8)                               System Manager's Manual                              TCPDUMP(8)

NAME
       tcpdump - dump traffic on a network

SYNOPSIS
       tcpdump [ -AbdDefhHIJKlLnNOpqStuUvxX# ] [ -B buffer_size ]
               [ -c count ] [ --count ] [ -C file_size ]
               [ -E spi@ipaddr algo:secret,...  ]
               [ -F file ] [ -G rotate_seconds ] [ -i interface ]
               [ --immediate-mode ] [ -j tstamp_type ] [ -m module ]
               [ -M secret ] [ --number ] [ --print ] [ -Q in|out|inout ]
               [ -r file ] [ -s snaplen ] [ -T type ] [ --version ]
               [ -V file ] [ -w file ] [ -W filecount ] [ -y datalinktype ]
               [ -z postrotate-command ] [ -Z user ]
               [ --time-stamp-precision=tstamp_precision ]
               [ --micro ] [ --nano ]
               [ expression ]

DESCRIPTION
       Tcpdump prints out a description of the contents of packets on a network interface that match the
       Boolean expression; the description is preceded by a time stamp, printed, by default,  as  hours,
       minutes, seconds, and fractions of a second since midnight.  It can also be run with the -w flag,
       which causes it to save the packet data to a file for later analysis, and/or with  the  -r  flag,
       which  causes  it to read from a saved packet file rather than to read packets from a network in‐
 Manual page tcpdump(8) line 1 (press h for help or q to quit)

Capturing traffic between my host and other host
`$ sudo tcpdump -n host 8.8.8.8`
> bash command not found
for each ping we get 2 packet records, 2 records because one packet is sent and one is sent back\
ICMP request =  ping request
64 bytes  = length of 64 [i guess that gives message of no loss]\
DNS uses port 53 by default\
tcpdump can show all dns requests machine sends `sudo tcpdump - n port 53`\

Windowsis on keerulisem, aga sarnast asja saab teha sellega: \
`netsh trace start capture=yes`

> You can use the following command if you want to specify the IP address.
> > `netsh trace start capture=yes IPv4.Address=X.X.X.X`

## Watching HTTP in tcpdump
Tcpdump can look at the packets that a machine uses to talk to a web server
We can use `nc` in linux to send out a request like `nc example.cet 80` while having `tcpdump -m port 80` to listen on it to examine  the packets
result is metadata, we dont actually see HTTP head requests
>assignment didnt qwork, didnt have permission and other one didnt do anything

## Analyzing tcpdump data
I should see my own machines IP. Is is on one on the other side of the bracket depending where from anf whwrw to packets were sent. Most in the tcp had 0 length.
> thats because before any data there needs to be a setup. 
> > first line of lenth with any value represents the total length of out request to server.

## Tcpdump packet quiz

response is the third one besauce outgoing ip is on the right side of > and has some value to it (321 length)

## Sequence diagrams
For network protocol illustration sequence diagram is useful
> http `GET... Content connection: keep-alive header` to -> server\
> server - > client : `contents of a main page`\
> Browser -> server `Get/favicon.ico`\
> Server - > client `favicon file`

Anything below is afferced by stuff above thats already happened\
One request on http level may be a shorthand for bunch of requests on a lower level

## Connection establishment

> What tcp does `how tcp does it`\
> communicate between 2 hosts ` ip layer (address+ routing)` [client has to send this message with port nr]\
> Multiple applications per host `port numbers`\
> [keeps track of data each point is sent and makes sure that other end has recieved it  <- TCP does that and makes sure that the application sees that data in order even if underlying network reorders the packets, it does that by putting a sequence nr to each packet]in-order delivery `sequence numbers`\
> Lossless delivery `Acknowledgement [each endpoint sends that to confirm reception, if a packet goes missing, end point will notice and resend it] + retransmission`\
> keeping connections distinct `random initial sequence numbers1\

packets have random nr --> seq nrs have random nrs. 

## Buffering
packets can be dropped, packets can be sent out of order although latter happens rarely. 
Each endpoint in the OS keeps space of memory that it fits incoming data into and it uses sequence nrs as byte positions. 

answer: the password is always swordfish

## TCP flags
flags ate in [] in TCPDUp and they always appear after port and address info. 
they are control bits on the tcp packet. 
Its a boolean value that is stored in memory as a single bit. if value = 1 flag is set, if 0 then cleared or unset. 

### six basic TCP flags
- SYN (synchronize) [S] — This packet is opening a new TCP session and contains a new initial sequence number.
- FIN (finish) [F] — This packet is used to close a TCP session normally. The sender is saying that they are finished sending, but they can still receive data from the other endpoint.
- PSH (push) [P] — This packet is the end of a chunk of application data, such as an HTTP request.
- RST (reset) [R] — This packet is a TCP error message; the sender has a problem and wants to reset (abandon) the session.
- ACK (acknowledge) [.] — This packet acknowledges that its sender has received data from the other endpoint. Almost every packet except the first SYN will have the ACK flag set.
- URG (urgent) [U] — This packet contains data that needs to be delivered to the application out-of-order. Not used in HTTP or most other current applications.

### three way handshake
The first packet sent to initiate a TCP session always has the SYN flag set. This initial SYN packet is what a client sends to a server to start opening a TCP connection. 
 [s]  This packet also contains a new, randomized sequence number (seq in tcpdump output).
 If the server accepts the connection, it sends a packet back that has the SYN and ACK flags, and acknowledges the initial SYN. This is the second packet in the sample data, with Flags [S.]. This contains a different initial sequence number.

(If the server doesn't want to accept the connection, it may not send anything at all. Or it may send a packet with the RST flag.)

Finally, the client acknowledges receiving the SYN|ACK packet by sending an ACK packet of its own.

This exchange of three packets is usually called the TCP three-way handshake. In addition to sequence numbers, the two endpoints also exchange other information used to set up the connection.

### Four way teardown
When either endpoint is done sending data into the connection, it can send a FIN packet to indicate that it is finished. The other endpoint will send an ACK to indicate that it has received the FIN.

In the example HTTP data, the client sends its FIN first, as soon as it is done sending the HTTP request. This is the first packet containing Flags [F.].

Eventually the other endpoint will be done sending as well, and will send a FIN of its own. Then the first endpoint will send an ACK.

 ### Inbetween
 In a long-running connection, there will be many packets exchanged back and forth. Some of them will contain application data; others may be only acknowledgments with no data (length 0). However, all TCP packets in a connection except the initial SYN will contain an acknowledgment of all the data that the sender has received so far. Therefore, they will all have the ACK flag set. (This is why tcpdump depicts the ACK flag with just a dot: it's really common.)
 
 ### ICMP and UDP dont have TCp flags
 If you look at tcpdump data for pings or basic DNS lookups, you will not see flags. This is because ping uses ICMP, and basic DNS lookups use UDP. These protocols do not have TCP flags or sequence numbers. [what is basic DNS lookup?]






------------------------------------
## Glossary
- Session - a TCP connection
- ack - acknowledgement for short
- OS - operating system
- flags - control bits on the tcp packet. Its a boolean value flag ack is like _roger_ in walkie talkie radio transmission
- [.] - ack flag (_roger_)

## Why do packets drop
Fast / slow / fast network can be reason. 
TCP doent send out data at full speed initially, its a a gradual process\
If packets are dropped, speed drops and then starts to rise again\
Routers drop packets and endpoints respond with decreased transfer speeds\
If router would keep up the speed and push everything throug, the connection wll time out (TTL will run out?)\
Above is TCP congestion control and thats very important in TCP

## TCP errors. 




