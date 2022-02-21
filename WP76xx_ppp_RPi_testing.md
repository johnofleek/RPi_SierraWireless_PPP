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

Chat script file names
```
chatDown
chatUp
pppWP76xx
```

**pppd settings**
Edit file *pppWP76xx* if required


**Copy the scripts**
To somewhere pppd can find them. For example

```
sudo cp pppWP76xx /etc/ppp/peers/
sudo cp chatUp /etc/chatscripts/
sudo cp chatDown /etc/chatscripts/
```





## Testing chat scripts

Standalone testing of chat scripts can be carried out like this. Where /dev/ttyUSB2 is the modem AT command port

```
sudo chat -v -f ./chatUp > /dev/ttyUSB2 < /dev/ttyUSB2
```




# Test on EE
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
AT+CGDCONT=3,"IP","everywhere"
AT+CGDCONT=4,"IP","everywhere"
AT+CGDCONT=5,"IP","everywhere"

AT+CGDCONT?


```

Manually add the PAP credentials
AT+CGAUTH=<cid>,<auth_prot>[, <userid>,<password>]
<auth_prot> (Required authentication type)
0—None. Username and password are not required.
1—PAP. Username and password accepted
2—CHAP. Username and password (secret) accepted

```
AT+CGAUTH=2,1,"eesecure","secure"
```

AT+CGAUTH=2,1,"secure","eesecure"

chat script establishes context and authentication prior to ppp negotiating the session by using

AT+CGACT=
[<state> [, <cid>
```
AT+CGACT=1,2
```

Check allocated address- this is not the same as the context address - I think for LTE it is the control address
```
AT+CGPADDR
```

Run with ee SIM using pppd as per three network example above
```
sudo pppd  /dev/ttyUSB2 record pptestee.txt call pppWP76xx
```

Connection testing worked OK  - results  
[record](./RC_pppRecords/pptestee.txt)  
[pcap](./RC_pppRecords/pptestee.pcap)  



# Test on EE with older modem FW

## WP76xx-1 SWI9X07H_00.08.19.00

Flashed with one click installer *RC76xx_Release9_BP6_GENERIC_test.exe*  


```
$ sudo pppd  /dev/ttyAMA0 115200 record pptestee_WP76.txt call pppWP76xx
```


Connection testing worked OK  - results  
[record](./RC_pppRecords/pptestee_08_19.txt)  
[pcap](./RC_pppRecords/pptestee_08_19.pcap)



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

# notes on FW





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


