## Scope
To see what happens when we use PPP with a Sierra Wireless RC7620 connected via USB UART to an RPi

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
chatDown
chatUp
pppRC7620
```

**pppd settings**
Edit file *pppRC7620* if required


**Copy the scripts**
To somewhere pppd can find them. For example

```
sudo cp pppRC7620 /etc/ppp/peers/
sudo cp chatUp /etc/chatscripts/
sudo cp chatDown /etc/chatscripts/
```



## Connect - Using a Three SIM

Try three first - doesn't require authentication

### set contexts

Terminal access RC7620 USB AT command port
```
sudo minicom -D /dev/ttyUSB2
```

Configure the contexts for the three network
```
AT+CGDCONT=1,"IP","three.co.uk"
AT+CGDCONT=2,"IP","three.co.uk"
AT+CGDCONT=3,"IP","three.co.uk"
AT+CGDCONT=4,"IP","three.co.uk"
AT+CGDCONT=5,"IP","three.co.uk"

AT+CGACT: 3,1
```
Exit minicom CTRL ALT x


### set authentication
TBD 

Not required for the three network - but note that the Pi Os is picking up PAP authentication anyway - this needs further investigation.

```
sent [PAP AuthReq id=0x1 user="raspberrypi" password=""]
```


### Make ppp connection
Make an IP Cellular WAN connections using pppd


```
sudo pppd  /dev/ttyUSB2 115200 call pppRC7620
```

### Make ppp connection and create a log that can be used with wireshark

For [examples](./RC_pppRecords) 
```
sudo pppd  /dev/ttyUSB2 record pptest.txt call pppRC7620
```

This logs the chat conversation and the ppp negotiation to pptest.txt. This is an append operation.

Wireshark can decode ppp negotiation.
In wireshark open the file created by pppd, wireshark will automatically decoded the ppp negotiation in a human readable format. 
Note that the record file also has the chat commands embedded.  



## Testing chat scripts

Standalone testing of chat scripts can be carried out like this. Where /dev/ttyUSB2 is the modem AT command port

```
sudo chat -v -f ./chatUp > /dev/ttyUSB2 < /dev/ttyUSB2
```




# Test on EE

APN: everywhere
Username: eesecure
Password: secure

Configure the contexts for the EE network
```
AT+CGDCONT=2,"IP","everywhere"
AT+CGDCONT=3,"IP","everywhere"
AT+CGDCONT=4,"IP","everywhere"
AT+CGDCONT=5,"IP","everywhere"

AT+CGDCONT?


```

Manually add the PAP credentials
```
OK AT+WPPP=1,3,"eesecure","secure"
```

chat script establishes context and authentication prior to ppp negotiating the session by using
```
AT+CGACT=3,1
```

Run with ee SIM using pppd as per three network example above
```
sudo pppd  /dev/ttyUSB2 record pptestee.txt call pppRC7620
```

Connection testing worked OK  - results  
[record](./RC_pppRecords/pptestee.txt  
[pcap](./RC_pppRecords/pptestee.pcap)  



# Test with older modem FW

## RC7620-1 SWI9X07H_00.08.19.00

Flashed with one click installer *RC76xx_Release9_BP6_GENERIC_test.exe*  

Modem reports

```
Model: RC7620
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


