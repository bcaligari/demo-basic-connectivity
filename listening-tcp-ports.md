## Listening TCP Ports

### Listening ports on a Linux host

`ss` is the maintained and supported tool on most Linux distributions for general socket statistics,
including listening TCP sockets.  To resolve all information, such as socket user, root privilages
may be required.

```
candle:~ # ss -tlnp
State   Recv-Q  Send-Q   Local Address:Port    Peer Address:Port Process                          
LISTEN  0       128            0.0.0.0:22           0.0.0.0:*     users:(("sshd",pid=2625,fd=3))  
LISTEN  0       10                   *:30002              *:*     users:(("socat",pid=7826,fd=5)) 
LISTEN  0       10                   *:30000              *:*     users:(("socat",pid=7664,fd=5)) 
LISTEN  0       128               [::]:22              [::]:*     users:(("sshd",pid=2625,fd=4)) 
```

### TCP port scan with nmap

[Nmap](https://nmap.org/) is an open source tool for network exploration and security auditing.

`nmap` has many options and used 'carelessly' can generate a lot of network noise that may trip
security alarms.

For a minimal TCP port scan to check whether a small number of TCP ports are open and listening
and where no added permissions are required on the scanning host the the following options can be
used:

* `-n` - Do not do any DNS resolutions
* `-Pn` - Treat all hosts as online -- skip host discovery
* `-sT` - Use system `connect()` to perform the scan
* `--reason` - Display the reason a port is in a particular state
* `-p` - Limit the scan to desired ports

```
sysop@arcadia:~> nmap -n --reason -Pn -sT 10.0.0.4 -p 22,30000,30001,30002,30003
Starting Nmap 7.94 ( https://nmap.org ) at 2024-09-20 12:13 UTC
Nmap scan report for 10.0.0.4
Host is up, received user-set (0.024s latency).

PORT      STATE    SERVICE        REASON
22/tcp    open     ssh            syn-ack
30000/tcp open     ndmps          syn-ack
30001/tcp closed   pago-services1 conn-refused
30002/tcp filtered pago-services2 no-response
30003/tcp filtered amicon-fpsu-ra no-response

Nmap done: 1 IP address (1 host up) scanned in 1.22 seconds
```

### Interpreting above results

#### open ports

We know that the host we are scanning is listening on 22, 30000, and 30002.

Ports 22 and 30000 were identified as 'open' with a reason of 'syn-ack' given.  This
means that we were able to connect to the port and completed the TCP 3 way handshake:

```
14:08:11.816312 IP 192.168.136.41.50418 > 10.0.0.4.30000: Flags [S], seq 1026107496, win 64240, options [mss 1452,sackOK,TS val 3846181660 ecr 0,nop,wscale 7], length 0
14:08:11.816350 IP 10.0.0.4.30000 > 192.168.136.41.50418: Flags [S.], seq 3018473357, ack 1026107497, win 65160, options [mss 1460,sackOK,TS val 853083560 ecr 3846181660,nop,wscale 7], length 0
14:08:11.839040 IP 192.168.136.41.50418 > 10.0.0.4.30000: Flags [.], ack 1, win 502, options [nop,nop,TS val 3846181683 ecr 853083560], length 0
14:08:11.839490 IP 192.168.136.41.50418 > 10.0.0.4.30000: Flags [R.], seq 1, ack 1, win 502, options [nop,nop,TS val 3846181683 ecr 853083560], length 0
```

#### closed ports

There is nothing listening on port 30001. `nmap` gives the 'closed' reason as 'conn-refused'.  In
this case this means that the remote host replied to our TCP SYN with a reset.

```
14:10:43.042030 IP 192.168.136.41.57066 > 10.0.0.4.30001: Flags [S], seq 2129400082, win 64240, options [mss 1452,sackOK,TS val 3846332886 ecr 0,nop,wscale 7], length 0
14:10:43.042067 IP 10.0.0.4.30001 > 192.168.136.41.57066: Flags [R.], seq 0, ack 2129400083, win 0, length 0
```

#### filtered ports

We know that the remote host is listening on 30002 but not 30003, yet we received a 'filtered' status
with a 'no-response' reason.  In this case our SYN packets were never received by the remote host.  This is
often the case if the port is blocked by a firewall.

#### caveats

In a TCP/IP network there is never a guarantee that the remote host, or any hop along the network for that
matter, are necessarily doing what we first assume they are doing.  The above three 'results' only hold if
all the actors on the network are playing nicely.
