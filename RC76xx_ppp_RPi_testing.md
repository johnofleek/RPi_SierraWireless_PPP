## Scope
To see what happens when we use PPP with a Sierra Wireless RC7620 connected via RC7620 composite USB device UART to an RPi  
Also  
To see what happens when we use PPPos with a Sierra Wireless RC7620 connected via RC7620 UART1 to the RPi /dev/ttyACM0 UART.  

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

 



## Testing chat scripts

Standalone testing of chat scripts can be carried out like this. Where /dev/ttyUSB2 is the modem AT command port

*USB - ttyUSB2*

```
sudo chat -v -f ./RC_chatScripts/chatUpRC7620 > /dev/ttyUSB2 < /dev/ttyUSB2
sudo chat -v -f ./RC_chatScripts/chatDownRC7620 > /dev/ttyUSB2 < /dev/ttyUSB2
```

*UART - ttyACM0*
Try without flow control and set baud rate
```
stty -F /dev/ttyAMA0 -crtscts
stty -F /dev/ttyAMA0  115200

sudo chat -v -f ./RC_chatScripts/chatUp > /dev/ttyAMA0 < /dev/ttyAMA0

sudo chat -v -f ./RC_chatScripts/chatDown > /dev/ttyAMA0 < /dev/ttyAMA0
```

# Test on EE

APN: everywhere
Username: eesecure
Password: secure

Configure the contexts for the EE network
```
AT+CGDCONT=1
AT+CGDCONT=2,"IP","everywhere"
AT+CGDCONT=3,"IP","everywhere"
AT+CGDCONT=4,"IP","everywhere"
AT+CGDCONT=5,"IP","everywhere"

AT+CGDCONT?


```

Manually add the PAP credentials
```
AT+WPPP=1,3,"eesecure","secure"
```

chat script establishes context and authentication prior to ppp negotiating the session by using
```
AT+CGACT=1,3
```

Run with ee SIM using pppd as per three network example above
```
sudo pppd  /dev/ttyUSB2 record pptestee.txt call pppRC7620
```

Connection testing worked OK  - results  
[record](./RC_pppRecords/pptestee.txt)  
[pcap](./RC_pppRecords/pptestee.pcap)  



# Test on EE with older modem FW RC7620-1 SWI9X07H_00.08.19.00

Flashed with one click installer *RC76xx_Release9_BP6_GENERIC_test.exe* 

This FW works ok with the USB serial port but with the UART serial port the PPP connection
 is made ok but following PPP disconnection the modem is in a broken state which can't be recovered
 without a AT!RESET 
 
 [some notes on forum](https://forum.sierrawireless.com/t/rc7620-pppos-uart-1-ppp-termination-causes-odd-modem-behaviour-requiring-reset-or-por/26323)


# Test on EE with later modem FW RC7620-1 SWI9X07H_00.08.24.00

Dial PPPos using USB interface
```
$ sudo pppd  /dev/ttyUSB2 record pptestee_08_19.txt call pppRC7620
```

Dial PPPos using Raspberry Pi physical UART /dev/ttyAMA0
```
$ sudo pppd  115200 /dev/ttyAMA0 record pptestee_08_19uart.txt call pppRC7620
```

This logs the chat conversation and the ppp negotiation to pptest.txt. This is an append operation.

Wireshark can decode ppp negotiation.
In wireshark open the file created by pppd, wireshark will automatically decoded the ppp negotiation in a human readable format. 
Note that the record file also has the chat commands embedded. 


Older USB connection testing worked OK  - results  
[record](./RC_pppRecords/pptestee_08_19.txt)  
[pcap](./RC_pppRecords/pptestee_08_19.pcap)



Modem reports

```
Model: RC7620-1
QTI baseline: MPSS.JO.2.0.2.c1.1-00073-9607_GENNS_PACK-1.416077.1.419248.1
Revision: SWI9X07H_00.08.24.00 cb6ed3 jenkins 2022/01/07 03:05:21
IMEI: 353635110154131
IMEI SV: 19
FSN: 7T1017851610B0
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
Connect: ppp1 <--> /dev/pts/2
sent [LCP ConfReq id=0x1 <asyncmap 0x0>]
rcvd [LCP ConfReq id=0x0 <asyncmap 0x0> <auth pap> <magic 0xf098bd1e> <pcomp> <accomp>]
sent [LCP ConfRej id=0x0 <magic 0xf098bd1e> <pcomp> <accomp>]
rcvd [LCP ConfAck id=0x1 <asyncmap 0x0>]
rcvd [LCP ConfReq id=0x1 <asyncmap 0x0> <auth pap>]
sent [LCP ConfAck id=0x1 <asyncmap 0x0> <auth pap>]
sent [LCP EchoReq id=0x0 magic=0x0]
sent [PAP AuthReq id=0x1 user="raspberrypi" password=""]
rcvd [LCP DiscReq id=0x2 magic=0xf098bd1e]
rcvd [LCP EchoRep id=0x0 magic=0xf098bd1e 00 00 00 00]
rcvd [PAP AuthAck id=0x1 ""]
PAP authentication succeeded
sent [IPCP ConfReq id=0x1 <addr 0.0.0.0> <ms-dns1 0.0.0.0> <ms-dns2 0.0.0.0>]
rcvd [IPCP ConfReq id=0x0]
sent [IPCP ConfNak id=0x0 <addr 0.0.0.0>]
rcvd [IPCP ConfNak id=0x1 <addr 100.69.130.124> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
sent [IPCP ConfReq id=0x2 <addr 100.69.130.124> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
rcvd [IPCP ConfReq id=0x1]
sent [IPCP ConfAck id=0x1]
rcvd [IPCP ConfAck id=0x2 <addr 100.69.130.124> <ms-dns1 109.249.185.228> <ms-dns2 109.249.185.229>]
Could not determine remote IP address: defaulting to 10.64.64.65
Script /etc/ppp/ip-pre-up started (pid 2958)
Script /etc/ppp/ip-pre-up finished (pid 2958), status = 0x0
replacing old default route to eth0 [10.10.10.1]
del old default route ioctl(SIOCDELRT): No such process(3)
local  IP address 100.69.130.124
remote IP address 10.64.64.65
primary   DNS address 109.249.185.228
secondary DNS address 109.249.185.229
```

# notes on FW

During testing we noticed that the modem doesn't offer a remote_IP_address 

See
https://linux.die.net/man/8/pppd
Options
<local_IP_address>:<remote_IP_address>

And a old but useful https://groups.google.com/g/comp.protocols.ppp/c/xo-sid_FoC0?pli=1

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


