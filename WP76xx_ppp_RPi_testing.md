## Scope
To see what happens when we use PPP with a Sierra Wireless WP76xx connected via Physical UART1 to an RPi /dev/ttyAMA0

Note that the the RPI serial port config was modified to disable Bluetooth so that
```
ls -l /dev/serial*
lrwxrwxrwx 1 root root  7 Feb 16 16:22 /dev/serial0 -> ttyAMA0
lrwxrwxrwx 1 root root  5 Feb 16 16:22 /dev/serial1 -> ttyS0
```

## Kit used
* RPi4
* WASP + WP7608 


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

### WP76xx modem FW version  
```
ati9
Manufacturer: Sierra Wireless, Incorporated
Model: WP7607
Revision: SWI9X07Y_02.37.03.00 73df45 jenkins 2020/04/08 10:59:14
IMEI: 359779080470764
IMEI SV: 14
FSN: V3048285141510
+GCAP: +CGSM
```


### Additional RPi software installed

```
sudo apt-get install ppp
```

### Scripts

**Chat script notes**

* chat doesn't like windows line termination - be careful to set your editor to Unix (LF)
* remember to disable the modems charater echo as echo will confuse your chat script logic


**pppd settings**
Edit file *pppWP76xx* if required


**Copy the scripts**
To somewhere pppd can find them. For example

```
sudo cp ./WP76_chatScripts/pppWP76xx /etc/ppp/peers/
sudo cp ./WP76_chatScripts/chatUpWP76xx /etc/chatscripts/
sudo cp ./WP76_chatScripts/chatDownWP76xx /etc/chatscripts/
```





## Manually testing chat scripts

Standalone testing of chat scripts can be carried out like this. Where /dev/ttyUSB2 is the modem AT command port

```
sudo chat -v -f ./chatUp > /dev/ttyUSB2 < /dev/ttyUSB2
```




# Test on EE
Summary of testing so far
* using CGACT in combo with  CGDCONT (and CGAUTH) on context 1&2 connect fails
* Using SIM (Things mobile) that doesn't require authentication - using context 1 or and default pppd PAP negotiation worked 
* testing with ee - when connected to LTE and using CGDCONT and CGAUTH - setting CGAUTH to anything the ppp negotiation succeeds


Check the WP76 UART mapping - use the WP USB OTG port or shell into the WP  and use /dev/ttyAT
```
at!MAPUART?
!MAPUART: 1,16
```

This means 
UART1 --> 1—AT command service (Note: Not available for UART2)
UART2 --> 16—Linux Console


AT command configuration via the RPi UART
```
sudo minicom -b 115200 -D /dev/ttyAMA0
```


Disable the UART sleep 
```
AT+ksleep=2
```

Set credentials for EE
```
APN: everywhere
Username: eesecure
Password: secure
```

Configure the contexts for the EE network
```
AT+CGDCONT=1
AT+CGDCONT=2,"IP","everywhere"
AT+CGDCONT=3
AT+CGDCONT=4
AT+CGDCONT=5

AT+CGDCONT?


```

Manually add the PAP credentials
```
AT+CGAUTH=<cid>,<auth_prot>[, <userid>,<password>]
<auth_prot> (Required authentication type)
0—None. Username and password are not required.
1—PAP. Username and password accepted
2—CHAP. Username and password (secret) accepted
```

```
AT+CGAUTH=2,1,"eesecure","secure"
```


chat script establishes context and authentication prior to ppp negotiating the session by using
```
AT+CGACT=
[<state> [, <cid>

AT+CGACT=1,2
```

Check allocated address- this is not the same as the context address - I think for LTE it is the control address
```
AT+CGPADDR
```

Note that 

```
AT+CGACT=0,2
ERROR

at+cgact?
+CGACT: 1,1
+CGACT: 2,0
```

and that context 2 has to be unconnected if using pppd. I have no ideas why
 as this is not the case for the RC76, HL and HL7802 modules






# Test on EE 

## WP76xx-1 SWI9X07H_00.08.19.00

Flashed with one click installer *RC76xx_Release9_BP6_GENERIC_test.exe*  


PPPos with ee SIM using pppd over USB serial port
```
sudo pppd  /dev/ttyUSB2 record pptestee_WP76.txt call pppWP76xx
```

PPPos with ee SIM using pppd over UART serial port and detach from shell
```
sudo pppd  /dev/ttyAMA0 115200  call pppWP76xx &
```

Kill pppd sessions
```
sudo killall pppd
```

Connection testing worked OK  - results  



Modem reports

```
Model: WP76xx
QTI baseline: MPSS.JO.2.0.2.c1.1-00073-9607_GENNS_PACK-1.416077.1.419248.1
Revision: SWI9X07H_00.08.19.00 725c0e jenkins 2021/09/08 14:36:45
IMEI: 353634110110227
IMEI SV: 18
FSN: 7Q0273850613B0
```

Check IP address
```
$ ifconfig ppp1
ppp1: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 100.68.11.121  netmask 255.255.255.255  destination 10.64.64.65
        ppp  txqueuelen 3  (Point-to-Point Protocol)
```

ping test works
```
$ ping -I ppp1 8.8.8.8
PING 8.8.8.8 (8.8.8.8) from 100.68.11.121 ppp1: 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=114 time=241 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=114 time=70.0 ms
64 bytes from 8.8.8.8: icmp_seq=3 ttl=114 time=39.1 ms
```

Syslog
```

```

# Notes on FW





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
sudo minicom -b 115200 -D /dev/ttyAMA0
```




## Check the IP interface

**Example response**

```
ifconfig ppp1



```

## Quick checks link is ok

```
pi@raspberrypi:~ $ ping -I ppp1 8.8.8.8



pi@raspberrypi:~ $ ping -I eth0  8.8.8.8

```

## Connect example on context 2 - EE SIM chauth set - but ppp PAP credentials not set to anything correct

```
Mar  1 11:36:15 raspberrypi pppd[2047]: pppd 2.4.9 started by pi, uid 0
Mar  1 11:36:16 raspberrypi chat[2049]: timeout set to 6 seconds
Mar  1 11:36:16 raspberrypi chat[2049]: send (AT^M)
Mar  1 11:36:16 raspberrypi chat[2049]: expect (OK)
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: OK
Mar  1 11:36:16 raspberrypi chat[2049]:  -- got it
Mar  1 11:36:16 raspberrypi chat[2049]: send (ATH0^M)
Mar  1 11:36:16 raspberrypi chat[2049]: expect (OK)
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: OK
Mar  1 11:36:16 raspberrypi chat[2049]:  -- got it
Mar  1 11:36:16 raspberrypi chat[2049]: send (ATE0^M)
Mar  1 11:36:16 raspberrypi chat[2049]: expect (OK)
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: OK
Mar  1 11:36:16 raspberrypi chat[2049]:  -- got it
Mar  1 11:36:16 raspberrypi chat[2049]: send (AT^M)
Mar  1 11:36:16 raspberrypi chat[2049]: abort on (BUSY)
Mar  1 11:36:16 raspberrypi chat[2049]: abort on (NO CARRIER)
Mar  1 11:36:16 raspberrypi chat[2049]: abort on (NO DIALTONE)
Mar  1 11:36:16 raspberrypi chat[2049]: abort on (NO ANSWER)
Mar  1 11:36:16 raspberrypi chat[2049]: abort on (DELAYED)
Mar  1 11:36:16 raspberrypi chat[2049]: expect (OK)
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: OK
Mar  1 11:36:16 raspberrypi chat[2049]:  -- got it
Mar  1 11:36:16 raspberrypi chat[2049]: send (ATDT*99***2#^M)
Mar  1 11:36:16 raspberrypi chat[2049]: expect (CONNECT)
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: ^M
Mar  1 11:36:16 raspberrypi chat[2049]: CONNECT
Mar  1 11:36:16 raspberrypi chat[2049]:  -- got it
Mar  1 11:36:16 raspberrypi chat[2049]: send (\d)
Mar  1 11:36:17 raspberrypi pppd[2047]: Script /usr/sbin/chat -v -f /etc/chatscripts/chatUpWP76xx finished (pid 2048), status = 0x0
Mar  1 11:36:17 raspberrypi pppd[2047]: Serial connection established.
Mar  1 11:36:17 raspberrypi pppd[2047]: using channel 4
Mar  1 11:36:17 raspberrypi pppd[2047]: Using interface ppp1
Mar  1 11:36:17 raspberrypi pppd[2047]: Connect: ppp1 <--> /dev/ttyAMA0
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [LCP ConfReq id=0x1 <asyncmap 0x0>]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [LCP ConfReq id=0x6 <asyncmap 0x0> <auth pap> <magic 0xd1324dde> <pcomp> <accomp>]
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [LCP ConfRej id=0x6 <magic 0xd1324dde> <pcomp> <accomp>]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [LCP ConfAck id=0x1 <asyncmap 0x0>]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [LCP ConfReq id=0x7 <asyncmap 0x0> <auth pap>]
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [LCP ConfAck id=0x7 <asyncmap 0x0> <auth pap>]
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [LCP EchoReq id=0x0 magic=0x0]
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [PAP AuthReq id=0x1 user="raspberrypi" password=""]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [LCP DiscReq id=0x8 magic=0xd1324dde]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [LCP EchoRep id=0x0 magic=0xd1324dde 00 00 00 00]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [PAP AuthAck id=0x1 ""]
Mar  1 11:36:18 raspberrypi pppd[2047]: PAP authentication succeeded
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [IPCP ConfReq id=0x2]
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [IPCP ConfNak id=0x2 <addr 0.0.0.0>]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [IPCP ConfNak id=0x1 <addr 10.71.157.107> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [IPCP ConfReq id=0x2 <addr 10.71.157.107> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [IPCP ConfReq id=0x3]
Mar  1 11:36:18 raspberrypi pppd[2047]: sent [IPCP ConfAck id=0x3]
Mar  1 11:36:18 raspberrypi pppd[2047]: rcvd [IPCP ConfAck id=0x2 <addr 10.71.157.107> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Mar  1 11:36:18 raspberrypi pppd[2047]: Could not determine remote IP address: defaulting to 10.64.64.65
Mar  1 11:36:18 raspberrypi pppd[2047]: Script /etc/ppp/ip-pre-up started (pid 2054)
Mar  1 11:36:18 raspberrypi pppd[2047]: Script /etc/ppp/ip-pre-up finished (pid 2054), status = 0x0
Mar  1 11:36:18 raspberrypi pppd[2047]: replacing old default route to eth0 [10.10.10.1]
Mar  1 11:36:18 raspberrypi pppd[2047]: del old default route ioctl(SIOCDELRT): No such process(3)
Mar  1 11:36:18 raspberrypi pppd[2047]: local  IP address 10.71.157.107
Mar  1 11:36:18 raspberrypi pppd[2047]: remote IP address 10.64.64.65
Mar  1 11:36:18 raspberrypi pppd[2047]: primary   DNS address 109.249.185.228
Mar  1 11:36:18 raspberrypi pppd[2047]: secondary DNS address 109.249.185.229
Mar  1 11:36:18 raspberrypi pppd[2047]: Script /etc/ppp/ip-up started (pid 2057)
Mar  1 11:36:19 raspberrypi avahi-daemon[369]: Got SIGHUP, reloading.
Mar  1 11:36:19 raspberrypi avahi-daemon[369]: No service file found in /etc/avahi/services.
Mar  1 11:36:19 raspberrypi avahi-daemon[369]: Failed to parse address 'fe80::4009:c4ff:fea0:df89%eth0', ignoring.
Mar  1 11:36:19 raspberrypi pppd[2047]: Script /etc/ppp/ip-up finished (pid 2057), status = 0x0
```

## radio state

```
at+cops?
+cops: 0,0,"3 UK Things Mobile",7

at!gstatus?
```