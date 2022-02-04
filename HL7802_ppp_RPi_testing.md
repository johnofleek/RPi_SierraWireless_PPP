## Scope
To see what happens when we use PPP with a Sierra Wireless RC7620 connected via RC7620 composite USB device UART to an RPi

## Kit used
* RPi4
* WASP + HL7802 


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

### HL7802 modem FW version  
Reflashing via the RPi is possible using sierra's sft method - note that AT port and boot port are the same. 
This FW was flashed using 
```
./sft -p /dev/ttyACM0 -g /dev/ttyACM0 -b 921600 -s AppFW_flash.bin sysHeader_backup.bin.alt1250 sysHeader.bin.alt1250 sysHeader_modem.bin.alt1250 u-boot.bin ue_lte_2g.fw partmap.bin ue_lte.fw2 

```

Reports ATi9
```
HL7802.4.6.9
HL78xx.4.6.9.4.RK_02_01_02_00_137.20210615
2021/06/15 21:00:02
IMEI-SV: 3594590901168914
```
### Modem config
Modem composition depends on the AT+KUSBCOMP setting

```
at+kusbcomp?
+KUSBCOMP: 1,1,2,3

ls -l /dev/ttyACM*
crw-rw---- 1 root dialout 166, 0 Feb  1 17:20 /dev/ttyACM0
crw-rw---- 1 root dialout 166, 1 Feb  1 17:19 /dev/ttyACM1
crw-rw---- 1 root dialout 166, 2 Feb  1 17:19 /dev/ttyACM2

```


 due to current (HL7802.4.6.9) ctx 3 not working
```
sudo cp pppHL7802 /etc/ppp/peers/
sudo cp chatUpHL7802 /etc/chatscripts/
sudo cp chatDownHL7802 /etc/chatscripts/
```



See RP76xx testing for more details regarding apps and settings


# Test with ee SIM



## Manual context activation test

Note for this modem cgdcont is able to display the modems IP address 



```
AT+CGDCONT=1
AT+CGDCONT=2,"IP","everywhere"

```
Only contexts 1 and 2 seem to work

```
AT+WPPP=1,2,"eesecure","secure"

AT+CGACT=1,2

at+cgdcont?
at+cgdcont?
+CGDCONT: 2,"IP","everywhere",,0,0,,0,,,,,,,,

ATE0
AT&W
```

Make a PPP connection - only ttyACM1 works - composition is set to ttyACM1 == AT_PPP
```
$ sudo pppd  /dev/ttyACM1 record HL7802_ppptestee.txt call pppHL7802
```


Connection testing worked OK  - results  
[record](.//.txt)  
[pcap](.//.pcap)



Check IP address
```
ifconfig ppp1

ppp1: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 10.135.80.191  netmask 255.255.255.255  destination 10.0.0.1
        ppp  txqueuelen 3  (Point-to-Point Protocol)


```

ping test works
```
ping -I ppp1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 10.135.80.191 ppp1: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=55 time=1270 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=55 time=468 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=55 time=284 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=55 time=285 ms

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
sudo minicom -D /dev/ttyACM1
```

## Standalone test chat scripts

Assumes default USB composition
Debug op is via syslog

```
sudo chat -v -f ./chatUpHL7802 > /dev/ttyACM1 < /dev/ttyACM1
```

```
sudo chat -v -f ./chatDownHL7802 > /dev/ttyACM1 < /dev/ttyACM1
```



## Check the IP interface

as above

## Actual command line session capture

```
sudo pppd  /dev/ttyACM1 record HL7802_ppptestee.txt call pppHL7802
```
Sys Log
```
Feb  4 12:00:46 raspberrypi pppd[3528]: pppd options in effect:
Feb  4 12:00:46 raspberrypi pppd[3528]: debug#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: nodetach#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: unit 1#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: dump#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: noauth#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: /dev/ttyACM1#011#011# (from command line)
Feb  4 12:00:46 raspberrypi pppd[3528]: nolock#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: connect /usr/sbin/chat -v -f /etc/chatscripts/chatUpHL7802#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: disconnect /usr/sbin/chat -v -f /etc/chatscripts/chatDownHL7802#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: record HL7802_ppptestee.txt#011#011# (from command line)
Feb  4 12:00:46 raspberrypi pppd[3528]: crtscts#011#011# (from /etc/ppp/options)
Feb  4 12:00:46 raspberrypi pppd[3528]: modem#011#011# (from /etc/ppp/options)
Feb  4 12:00:46 raspberrypi pppd[3528]: noaccomp#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: asyncmap 0#011#011# (from /etc/ppp/options)
Feb  4 12:00:46 raspberrypi pppd[3528]: nomagic#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: nopcomp#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: lcp-echo-failure 4#011#011# (from /etc/ppp/options)
Feb  4 12:00:46 raspberrypi pppd[3528]: lcp-echo-interval 30#011#011# (from /etc/ppp/options)
Feb  4 12:00:46 raspberrypi pppd[3528]: hide-password#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: novj#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: ipcp-accept-local#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: ipcp-accept-remote#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: noipdefault#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: defaultroute#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: replacedefaultroute#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: usepeerdns#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: :#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: noipv6#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: noccp#011#011# (from /etc/ppp/peers/pppHL7802)
Feb  4 12:00:46 raspberrypi pppd[3528]: noipx#011#011# (from /etc/ppp/options)
Feb  4 12:00:46 raspberrypi pppd[3528]: pppd 2.4.9 started by pi, uid 0
Feb  4 12:00:47 raspberrypi chat[3531]: timeout set to 6 seconds
Feb  4 12:00:47 raspberrypi chat[3531]: abort on (BUSY)
Feb  4 12:00:47 raspberrypi chat[3531]: abort on (NO CARRIER)
Feb  4 12:00:47 raspberrypi chat[3531]: abort on (NO DIALTONE)
Feb  4 12:00:47 raspberrypi chat[3531]: abort on (NO ANSWER)
Feb  4 12:00:47 raspberrypi chat[3531]: abort on (DELAYED)
Feb  4 12:00:47 raspberrypi chat[3531]: send (+++)
Feb  4 12:00:47 raspberrypi chat[3531]: send (\d)
Feb  4 12:00:48 raspberrypi chat[3531]: send (ATH^M)
Feb  4 12:00:48 raspberrypi chat[3531]: send (ATE0^M)
Feb  4 12:00:48 raspberrypi chat[3531]: send (AT^M)
Feb  4 12:00:48 raspberrypi chat[3531]: expect (OK)
Feb  4 12:00:48 raspberrypi chat[3531]: ^M
Feb  4 12:00:48 raspberrypi chat[3531]: ERROR^M
Feb  4 12:00:48 raspberrypi chat[3531]: ^M
Feb  4 12:00:48 raspberrypi chat[3531]: OK
Feb  4 12:00:48 raspberrypi chat[3531]:  -- got it
Feb  4 12:00:48 raspberrypi chat[3531]: send (AT+CGACT=0,2^M)
Feb  4 12:00:49 raspberrypi chat[3531]: send (\d)
Feb  4 12:00:50 raspberrypi chat[3531]: expect (OK)
Feb  4 12:00:50 raspberrypi chat[3531]: ^M
Feb  4 12:00:50 raspberrypi chat[3531]: ^M
Feb  4 12:00:50 raspberrypi chat[3531]: OK
Feb  4 12:00:50 raspberrypi chat[3531]:  -- got it
Feb  4 12:00:50 raspberrypi chat[3531]: send (AT+CGACT=1,2^M)
Feb  4 12:00:50 raspberrypi chat[3531]: expect (OK)
Feb  4 12:00:50 raspberrypi chat[3531]: ^M
Feb  4 12:00:50 raspberrypi chat[3531]: ^M
Feb  4 12:00:50 raspberrypi chat[3531]: OK
Feb  4 12:00:50 raspberrypi chat[3531]:  -- got it
Feb  4 12:00:50 raspberrypi chat[3531]: send (ATDT*99***2#^M)
Feb  4 12:00:50 raspberrypi chat[3531]: expect (CONNECT)
Feb  4 12:00:50 raspberrypi chat[3531]: ^M
Feb  4 12:00:51 raspberrypi chat[3531]: ^M
Feb  4 12:00:51 raspberrypi chat[3531]: OK^M
Feb  4 12:00:51 raspberrypi chat[3531]: ^M
Feb  4 12:00:51 raspberrypi chat[3531]: CONNECT
Feb  4 12:00:51 raspberrypi chat[3531]:  -- got it
Feb  4 12:00:51 raspberrypi chat[3531]: send (\d)
Feb  4 12:00:52 raspberrypi pppd[3528]: Script /usr/sbin/chat -v -f /etc/chatscripts/chatUpHL7802 finished (pid 3530), status = 0x0
Feb  4 12:00:52 raspberrypi pppd[3528]: Serial connection established.
Feb  4 12:00:52 raspberrypi pppd[3528]: using channel 11
Feb  4 12:00:52 raspberrypi pppd[3528]: Using interface ppp1
Feb  4 12:00:52 raspberrypi pppd[3528]: Connect: ppp1 <--> /dev/pts/2
Feb  4 12:00:53 raspberrypi pppd[3528]: sent [LCP ConfReq id=0x1 <asyncmap 0x0>]
Feb  4 12:00:53 raspberrypi pppd[3528]: rcvd [LCP ConfAck id=0x1 <asyncmap 0x0>]
Feb  4 12:00:56 raspberrypi pppd[3528]: sent [LCP ConfReq id=0x1 <asyncmap 0x0>]
Feb  4 12:00:56 raspberrypi pppd[3528]: rcvd [LCP ConfAck id=0x1 <asyncmap 0x0>]
Feb  4 12:00:57 raspberrypi pppd[3528]: rcvd [LCP ConfReq id=0x1 <asyncmap 0x0> <magic 0xa6bb1df3> <pcomp> <accomp>]
Feb  4 12:00:57 raspberrypi pppd[3528]: sent [LCP ConfRej id=0x1 <magic 0xa6bb1df3> <pcomp> <accomp>]
Feb  4 12:00:57 raspberrypi pppd[3528]: rcvd [LCP ConfReq id=0x2 <asyncmap 0x0>]
Feb  4 12:00:57 raspberrypi pppd[3528]: sent [LCP ConfAck id=0x2 <asyncmap 0x0>]
Feb  4 12:00:57 raspberrypi pppd[3528]: sent [LCP EchoReq id=0x0 magic=0x0]
Feb  4 12:00:57 raspberrypi pppd[3528]: sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
Feb  4 12:00:57 raspberrypi pppd[3528]: rcvd [IPCP ConfReq id=0x1 <compress VJ 0f 01> <addr 10.0.0.1> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Feb  4 12:00:57 raspberrypi pppd[3528]: sent [IPCP ConfRej id=0x1 <compress VJ 0f 01> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Feb  4 12:00:57 raspberrypi pppd[3528]: rcvd [IPV6CP ConfReq id=0x1]
Feb  4 12:00:57 raspberrypi pppd[3528]: Unsupported protocol 'IPv6 Control Protocol' (0x8057) received
Feb  4 12:00:57 raspberrypi pppd[3528]: sent [LCP ProtRej id=0x2 80 57 01 01 00 04]
Feb  4 12:00:57 raspberrypi pppd[3528]: rcvd [LCP EchoRep id=0x0 magic=0x0]
Feb  4 12:00:57 raspberrypi pppd[3528]: rcvd [IPCP ConfNak id=0x1 <addr 10.133.54.20> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Feb  4 12:00:57 raspberrypi pppd[3528]: sent [IPCP ConfReq id=0x2 <addr 10.133.54.20> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Feb  4 12:00:57 raspberrypi pppd[3528]: rcvd [IPCP ConfReq id=0x2 <addr 10.0.0.1>]
Feb  4 12:00:57 raspberrypi pppd[3528]: sent [IPCP ConfAck id=0x2 <addr 10.0.0.1>]
Feb  4 12:00:57 raspberrypi pppd[3528]: rcvd [IPCP ConfAck id=0x2 <addr 10.133.54.20> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Feb  4 12:00:57 raspberrypi pppd[3528]: Script /etc/ppp/ip-pre-up started (pid 3536)
Feb  4 12:00:57 raspberrypi pppd[3528]: Script /etc/ppp/ip-pre-up finished (pid 3536), status = 0x0
Feb  4 12:00:57 raspberrypi pppd[3528]: replacing old default route to eth0 [10.10.10.1]
Feb  4 12:00:57 raspberrypi pppd[3528]: del old default route ioctl(SIOCDELRT): No such process(3)
Feb  4 12:00:57 raspberrypi pppd[3528]: local  IP address 10.133.54.20
Feb  4 12:00:57 raspberrypi pppd[3528]: remote IP address 10.0.0.1
Feb  4 12:00:57 raspberrypi pppd[3528]: primary   DNS address 109.249.185.228
Feb  4 12:00:57 raspberrypi pppd[3528]: secondary DNS address 109.249.185.229
Feb  4 12:00:57 raspberrypi pppd[3528]: Script /etc/ppp/ip-up started (pid 3539)
Feb  4 12:00:57 raspberrypi avahi-daemon[361]: Got SIGHUP, reloading.
Feb  4 12:00:57 raspberrypi avahi-daemon[361]: No service file found in /etc/avahi/services.
Feb  4 12:00:57 raspberrypi avahi-daemon[361]: Failed to parse address 'fe80::4009:c4ff:fea0:df89%eth0', ignoring.
Feb  4 12:00:57 raspberrypi pppd[3528]: Script /etc/ppp/ip-up finished (pid 3539), status = 0x0
```