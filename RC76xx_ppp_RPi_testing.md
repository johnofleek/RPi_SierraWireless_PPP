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

For [examples](./pppRecords) 
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





