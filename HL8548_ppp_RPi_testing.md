## Scope
To see what happens when we use PPP with a Sierra Wireless RC7620 connected via RC7620 composite USB device UART to an RPi

## Kit used
* RPi4
* WASP + WP7620 


### Versions used for testing (Latest available 2022-01)
### RPi 4 OS  
```
$ uname -a
Linux raspberrypi 5.10.63-v7l+ #1459 SMP Wed Oct 6 16:41:57 BST 2021 armv7l GNU/Linu

$ cat /etc/os-release
PRETTY_NAME="Raspbian GNU/Linux 11 (bullseye)"
NAME="Raspbian GNU/Linux"
VERSION_ID="11"
```

### HL8548 modem FW version  
```
RHL85xx.5.5.24.0.201704131036.x6250_4
2017/04/13 11:03:56
```

See RP76xx testing for more details regarding apps and settings


# Test with ee SIM

Modem reports

```
RHL85xx.5.5.24.0.201704131036.x6250_4
2017/04/13 11:03:56


ls -l /dev/ttyACM*
crw-rw---- 1 root dialout 166, 0 Feb  1 09:58 /dev/ttyACM0
crw-rw---- 1 root dialout 166, 1 Feb  1 09:45 /dev/ttyACM1
crw-rw---- 1 root dialout 166, 2 Feb  1 09:45 /dev/ttyACM2
crw-rw---- 1 root dialout 166, 3 Feb  1 09:45 /dev/ttyACM3
crw-rw---- 1 root dialout 166, 4 Feb  1 09:45 /dev/ttyACM4
crw-rw---- 1 root dialout 166, 5 Feb  1 09:45 /dev/ttyACM5
crw-rw---- 1 root dialout 166, 6 Feb  1 09:45 /dev/ttyACM6
```

## Manual context activation test

Note for this modem cgdcont is able to display the modems IP address 

```
AT+WPPP=1,3,"eesecure","secure"

AT+CGACT=1,3

at+cgdcont?
+CGDCONT: 2,"IP","everywhere","0.0.0.0",0,0
+CGDCONT: 3,"IP","everywhere","100.106.251.48",0,0
+CGDCONT: 4,"IP","everywhere","0.0.0.0",0,0
+CGDCONT: 5,"IP","everywhere","0.0.0.0",0,0
```

Make a PPP connection
```
$ sudo pppd  /dev/ttyACM0 record HL85_ppptestee.txt call pppRC7620
```


Connection testing worked OK  - results  
[record](./HL8548_pppRecords/HL85_ppptestee.txt)  
[pcap](./HL8548_pppRecords/HL85_ppptestee.pcap)



Check IP address
```
$ ifconfig ppp1
ppp1: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 100.106.251.48  netmask 255.255.255.255  destination 192.200.1.21

```

ping test works
```
$ ping -I ppp1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 100.106.251.48 ppp1: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=1109 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=415 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=135 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=114 time=74.2 ms

```


Syslog
```
Connect: ppp1 <--> /dev/pts/2
rcvd [LCP ConfReq id=0x1 <asyncmap 0x0> <auth chap MD5> <magic 0x52181044> <pcomp> <accomp>]
sent [LCP ConfReq id=0x1 <asyncmap 0x0>]
sent [LCP ConfRej id=0x1 <magic 0x52181044> <pcomp> <accomp>]
rcvd [LCP ConfAck id=0x1 <asyncmap 0x0>]
rcvd [LCP ConfReq id=0x2 <asyncmap 0x0> <auth chap MD5>]
sent [LCP ConfNak id=0x2 <auth pap>]
rcvd [LCP ConfReq id=0x3 <asyncmap 0x0> <auth pap>]
sent [LCP ConfAck id=0x3 <asyncmap 0x0> <auth pap>]
sent [LCP EchoReq id=0x0 magic=0x0]
sent [PAP AuthReq id=0x1 user="raspberrypi" password=""]
rcvd [LCP EchoRep id=0x0 magic=0x0]
rcvd [PAP AuthAck id=0x1 ""]
PAP authentication succeeded
sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
rcvd [IPCP ConfReq id=0x1]
sent [IPCP ConfNak id=0x1 <addr 0.0.0.0>]
rcvd [IPCP ConfNak id=0x1 <addr 100.106.251.48> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
sent [IPCP ConfReq id=0x2 <addr 100.106.251.48> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
rcvd [IPCP ConfReq id=0x2 <addr 192.200.1.21>]
sent [IPCP ConfAck id=0x2 <addr 192.200.1.21>]
rcvd [IPCP ConfAck id=0x2 <addr 100.106.251.48> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Script /etc/ppp/ip-pre-up started (pid 1647)
Script /etc/ppp/ip-pre-up finished (pid 1647), status = 0x0
replacing old default route to eth0 [10.10.10.1]
del old default route ioctl(SIOCDELRT): No such process(3)
local  IP address 100.106.251.48
remote IP address 192.200.1.21
primary   DNS address 109.249.185.228
secondary DNS address 109.249.185.229
```





# Debugging
### Check script execution - do this in another shell terminal
```
tail -f /var/log/syslog
```

## Talk to the modem directly from the raspberry pi
For details on the modem AT command manual at source.sierrawireless.com
UART example

```
sudo apt-get install minicom
sudo minicom -D /dev/ttyACM0
```




## Check the IP interface

**Example response**

```
ifconfig ppp1

ppp1: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 172.26.168.84  netmask 255.255.255.255  destination 172.26.168.84
        ppp  txqueuelen 3  (Point-to-Point Protocol)
        RX packets 4  bytes 58 (58.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4  bytes 64 (64.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```

## Quick checks link is ok

```
pi@raspberrypi:~ $ ping -I ppp1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 100.69.130.124 ppp1: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=318 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=67.1 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=35.3 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=113 time=72.7 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=113 time=61.7 ms


pi@raspberrypi:~ $ ping -I eth0  8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.10.10.100 eth0: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=113 time=61.5 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=113 time=39.5 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=113 time=37.7 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=113 time=34.8 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=113 time=74.6 ms
```


