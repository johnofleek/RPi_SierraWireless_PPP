## Scope
To see what happens when we use PPPos with a Sierra Wireless RC7620 connected via RC7620 UART1 to the RPi /dev/ttyACM0 UART.

## Purpose
Test the process with a Things Mobile SIM which does not require authentication.
We leave context 1 not configured - note that context 1 connects anyway - is this due to LTE host network signaling requirements?

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

### RC7620 modem FW version  
```
Manufacturer: Sierra Wireless, Incorporated
Model: RC7620
QTI baseline: MPSS.JO.2.0.2.c1.1-00073-9607_GENNS_PACK-1.416077.1.419248.1
Revision: SWI9X07H_00.08.24.00 cb6ed3 jenkins 2022/01/07 03:05:21
IMEI: 353634110110227
IMEI SV: 19
FSN: 7Q0273850613B0
+GCAP: +CGSM

From One-click RC76xx_Release9_BP10_GENERIC_test.exe
which contains RC76xx_Release9_BP10_GENERIC_test.spk
```


### Additional RPi software installed

```
sudo apt-get install ppp
```

### Scripts

**Chat script notes**

* chat doesn't like windows line termination - be careful to set your editor to Unix (LF)
* remember to disable the modems charater echo as echo will confuse your chat script logic

Chat script file names
```
chatDownRC7620
chatUpRC7620
pppRC7620
```

**pppd settings**
Edit file *pppRC7620* if required


**Copy the scripts**
To somewhere pppd can find them. For example

```
sudo cp ./RC_chatScripts/pppRC7620 /etc/ppp/peers/
sudo cp ./RC_chatScripts/chatUpRC7620 /etc/chatscripts/
sudo cp ./RC_chatScripts/chatDownRC7620 /etc/chatscripts/
```

Note that chatUpRC7620 now tests to see if the modem responds to AT commands - if it doesn't 
 the script tries switching to command mode and then hangs up.
 This was added because terminating pppd via the UART interface doesn't hang up the modem.
 I suspect that using DTR to hangup may fix the issue but haven't tested this


#### stop pppd if nodetach isn't enabled

The method I have found (so far) is
```
sudo killall pppd
```

 


# Manual connection Test on Things Mobile

APN: TM
Username: 
Password: 

Configure the contexts for the EE network
```
AT+CGDCONT=1
AT+CGDCONT=2
AT+CGDCONT=3,"IP","TM"
AT+CGDCONT=4
AT+CGDCONT=5

at+cgdcont?
+CGDCONT: 1,"IPV4V6","","0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0",0,0,0,0
+CGDCONT: 2,"IPV4V6","ims","0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0",0,0,0,0
+CGDCONT: 3,"IP","TM","0.0.0.0",0,0,0,0
```

Manually add the PAP credentials
```
AT+WPPP=0,3

at+wppp?

+WPPP: 0,1,"",""
+WPPP: 0,2,"",""
+WPPP: 0,3,"","
```

chat script establishes context and authentication prior to ppp negotiating the session by using
```
AT+CGACT=1,3
OK

at+cgact?
+CGACT: 1,1
+CGACT: 2,0
+CGACT: 3,1
```


# Test on Things Mobile with modem FW RC7620-1 SWI9X07H_00.08.24.00

Dial PPPos using Raspberry Pi physical UART /dev/ttyAMA0
```
$ sudo pppd  115200 /dev/ttyAMA0 call pppRC7620
```


## Example connection

```
Feb 28 09:42:25 raspberrypi pppd[1442]: unrecognized option 'pppRC7620'
Feb 28 09:42:43 raspberrypi kernel: [ 1061.416958] PPP generic driver version 2.4.2
Feb 28 09:42:43 raspberrypi pppd[1446]: pppd options in effect:
Feb 28 09:42:43 raspberrypi pppd[1446]: debug#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: nodetach#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: unit 1#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: dump#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: noauth#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: /dev/ttyAMA0#011#011# (from command line)
Feb 28 09:42:43 raspberrypi pppd[1446]: 115200#011#011# (from command line)
Feb 28 09:42:43 raspberrypi pppd[1446]: nolock#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: connect /usr/sbin/chat -v -f /etc/chatscripts/chatUpRC7620#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: disconnect /usr/sbin/chat -v -f /etc/chatscripts/chatDownRC7620#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: nocrtscts#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: modem#011#011# (from /etc/ppp/options)
Feb 28 09:42:43 raspberrypi pppd[1446]: noaccomp#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: asyncmap 0#011#011# (from /etc/ppp/options)
Feb 28 09:42:43 raspberrypi pppd[1446]: nomagic#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: nopcomp#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: lcp-echo-failure 4#011#011# (from /etc/ppp/options)
Feb 28 09:42:43 raspberrypi pppd[1446]: lcp-echo-interval 30#011#011# (from /etc/ppp/options)
Feb 28 09:42:43 raspberrypi pppd[1446]: show-password#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: novj#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: ipcp-accept-local#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: ipcp-accept-remote#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: noipdefault#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: defaultroute#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: replacedefaultroute#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: usepeerdns#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: :#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: noipv6#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: noccp#011#011# (from /etc/ppp/peers/pppRC7620)
Feb 28 09:42:43 raspberrypi pppd[1446]: noipx#011#011# (from /etc/ppp/options)
Feb 28 09:42:43 raspberrypi pppd[1446]: pppd 2.4.9 started by pi, uid 0
Feb 28 09:42:44 raspberrypi chat[1453]: timeout set to 6 seconds
Feb 28 09:42:44 raspberrypi chat[1453]: send (AT^M)
Feb 28 09:42:44 raspberrypi chat[1453]: expect (OK)
Feb 28 09:42:44 raspberrypi chat[1453]: AT^M^M
Feb 28 09:42:44 raspberrypi chat[1453]: OK
Feb 28 09:42:44 raspberrypi chat[1453]:  -- got it
Feb 28 09:42:44 raspberrypi chat[1453]: send (ATH0^M)
Feb 28 09:42:44 raspberrypi chat[1453]: expect (OK)
Feb 28 09:42:44 raspberrypi chat[1453]: ^M
Feb 28 09:42:44 raspberrypi chat[1453]: ATH0^M^M
Feb 28 09:42:44 raspberrypi chat[1453]: OK
Feb 28 09:42:44 raspberrypi chat[1453]:  -- got it
Feb 28 09:42:44 raspberrypi chat[1453]: send (ATE0^M)
Feb 28 09:42:44 raspberrypi chat[1453]: expect (OK)
Feb 28 09:42:44 raspberrypi chat[1453]: ^M
Feb 28 09:42:44 raspberrypi chat[1453]: ATE0^M^M
Feb 28 09:42:44 raspberrypi chat[1453]: OK
Feb 28 09:42:44 raspberrypi chat[1453]:  -- got it
Feb 28 09:42:44 raspberrypi chat[1453]: send (AT^M)
Feb 28 09:42:44 raspberrypi chat[1453]: expect (OK)
Feb 28 09:42:44 raspberrypi chat[1453]: ^M
Feb 28 09:42:44 raspberrypi chat[1453]: ^M
Feb 28 09:42:44 raspberrypi chat[1453]: OK
Feb 28 09:42:44 raspberrypi chat[1453]:  -- got it
Feb 28 09:42:44 raspberrypi chat[1453]: send (AT+CGACT=1,3^M)
Feb 28 09:42:44 raspberrypi chat[1453]: abort on (BUSY)
Feb 28 09:42:44 raspberrypi chat[1453]: abort on (NO CARRIER)
Feb 28 09:42:44 raspberrypi chat[1453]: abort on (NO DIALTONE)
Feb 28 09:42:44 raspberrypi chat[1453]: abort on (NO ANSWER)
Feb 28 09:42:44 raspberrypi chat[1453]: abort on (DELAYED)
Feb 28 09:42:44 raspberrypi chat[1453]: expect (OK)
Feb 28 09:42:44 raspberrypi chat[1453]: ^M
Feb 28 09:42:44 raspberrypi chat[1453]: ^M
Feb 28 09:42:44 raspberrypi chat[1453]: OK
Feb 28 09:42:44 raspberrypi chat[1453]:  -- got it
Feb 28 09:42:44 raspberrypi chat[1453]: send (ATDT*99***3#^M)
Feb 28 09:42:44 raspberrypi chat[1453]: expect (CONNECT)
Feb 28 09:42:44 raspberrypi chat[1453]: ^M
Feb 28 09:42:44 raspberrypi chat[1453]: ^M
Feb 28 09:42:44 raspberrypi chat[1453]: CONNECT
Feb 28 09:42:44 raspberrypi chat[1453]:  -- got it
Feb 28 09:42:44 raspberrypi chat[1453]: send (\d)
Feb 28 09:42:45 raspberrypi pppd[1446]: Script /usr/sbin/chat -v -f /etc/chatscripts/chatUpRC7620 finished (pid 1452), status = 0x0
Feb 28 09:42:45 raspberrypi pppd[1446]: Serial connection established.
Feb 28 09:42:45 raspberrypi pppd[1446]: using channel 1
Feb 28 09:42:45 raspberrypi pppd[1446]: Using interface ppp1
Feb 28 09:42:45 raspberrypi pppd[1446]: Connect: ppp1 <--> /dev/ttyAMA0
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [LCP ConfReq id=0x1 <asyncmap 0x0>]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [LCP ConfReq id=0x0 <asyncmap 0x0> <auth chap MD5> <magic 0x7f9ec41c> <pcomp> <accomp>]
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [LCP ConfRej id=0x0 <magic 0x7f9ec41c> <pcomp> <accomp>]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [LCP ConfAck id=0x1 <asyncmap 0x0>]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [LCP ConfReq id=0x1 <asyncmap 0x0> <auth chap MD5>]
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [LCP ConfNak id=0x1 <auth pap>]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [LCP ConfReq id=0x2 <asyncmap 0x0> <auth pap>]
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [LCP ConfAck id=0x2 <asyncmap 0x0> <auth pap>]
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [LCP EchoReq id=0x0 magic=0x0]
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [PAP AuthReq id=0x1 user="raspberrypi" password=""]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [LCP DiscReq id=0x3 magic=0x7f9ec41c]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [LCP EchoRep id=0x0 magic=0x7f9ec41c 00 00 00 00]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [PAP AuthAck id=0x1 ""]
Feb 28 09:42:46 raspberrypi pppd[1446]: PAP authentication succeeded
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [IPCP ConfReq id=0x0]
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [IPCP ConfNak id=0x0 <addr 0.0.0.0>]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [IPCP ConfNak id=0x1 <addr 10.109.77.68> <ms-dns1 8.8.8.8> <ms-dns2 8.8.4.4>]
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [IPCP ConfReq id=0x2 <addr 10.109.77.68> <ms-dns1 8.8.8.8> <ms-dns2 8.8.4.4>]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [IPCP ConfReq id=0x1]
Feb 28 09:42:46 raspberrypi pppd[1446]: sent [IPCP ConfAck id=0x1]
Feb 28 09:42:46 raspberrypi pppd[1446]: rcvd [IPCP ConfAck id=0x2 <addr 10.109.77.68> <ms-dns1 8.8.8.8> <ms-dns2 8.8.4.4>]
Feb 28 09:42:46 raspberrypi pppd[1446]: Could not determine remote IP address: defaulting to 10.64.64.65
Feb 28 09:42:46 raspberrypi pppd[1446]: Script /etc/ppp/ip-pre-up started (pid 1457)
Feb 28 09:42:46 raspberrypi pppd[1446]: Script /etc/ppp/ip-pre-up finished (pid 1457), status = 0x0
Feb 28 09:42:46 raspberrypi pppd[1446]: replacing old default route to eth0 [10.10.10.1]
Feb 28 09:42:46 raspberrypi pppd[1446]: del old default route ioctl(SIOCDELRT): No such process(3)
Feb 28 09:42:46 raspberrypi pppd[1446]: local  IP address 10.109.77.68
Feb 28 09:42:46 raspberrypi pppd[1446]: remote IP address 10.64.64.65
Feb 28 09:42:46 raspberrypi pppd[1446]: primary   DNS address 8.8.8.8
Feb 28 09:42:46 raspberrypi pppd[1446]: secondary DNS address 8.8.4.4
Feb 28 09:42:46 raspberrypi pppd[1446]: Script /etc/ppp/ip-up started (pid 1460)
Feb 28 09:42:47 raspberrypi avahi-daemon[366]: Got SIGHUP, reloading.
Feb 28 09:42:47 raspberrypi avahi-daemon[366]: No service file found in /etc/avahi/services.
Feb 28 09:42:47 raspberrypi avahi-daemon[366]: Failed to parse address 'fe80::4009:c4ff:fea0:df89%eth0', ignoring.
Feb 28 09:42:47 raspberrypi pppd[1446]: Script /etc/ppp/ip-up finished (pid 1460), status = 0x0
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
        inet 10.109.77.68  netmask 255.255.255.255  destination 10.64.64.65

```

## Quick checks link is ok

```
pi@raspberrypi:~ $ ping -I ppp1 8.8.8.8
64 bytes from 8.8.8.8: icmp_seq=1 ttl=112 time=164 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=112 time=162 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=112 time=143 ms
64 bytes from 8.8.8.8: icmp_seq=4 ttl=112 time=174 ms
64 bytes from 8.8.8.8: icmp_seq=5 ttl=112 time=162 ms
64 bytes from 8.8.8.8: icmp_seq=6 ttl=112 time=177 ms

pi@raspberrypi:~ $ ping -I eth0  8.8.8.8

```

## radio state

```
at+cops?
+cops: 0,0,"3 UK Things Mobile",7

at!gstatus?
!GSTATUS:
Current Time:  1426             Temperature: 26
Modem Mitigate Level: 0         ModemProc Mitigate Level: 0
Reset Counter: 1                Mode:        ONLINE
System mode:   LTE              PS state:    Attached
IMS reg state: NOT REGISTERED   IMS mode:    Normal
IMS Srv State: NO SMS,NO VoIP
LTE band:      B3               LTE bw:      15 MHz
LTE Rx chan:   1392             LTE Tx chan: 19392
LTE CA state:  INACTIVE
EMM state:     Registered       Normal Service
RRC state:     RRC Connected

PCC RxM RSSI:  -72              RSRP (dBm):  -105
PCC RxD RSSI:  -94              RSRP (dBm):  -132
Tx Power:      --               TAC:         0E62 (3682)
RSRQ (dB):     -14              Cell ID:     002AC602 (2803202)
SINR (dB):      8.4

OK
```
