## Scope
To see what happens when we use PPP with a Sierra Wireless RC7620 connected via USB UART to an RPi

## Kit used
* RPi4
* WASP + WP7620 


## Versions used for testing (Latest available 2022-01)
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


## Additional RPi software installed

```
sudo apt-get install ppp
```

## SIM - Three internet settings

Try three first - doesn't require authentication

### set contexts

Terminal access RC7620 USB AT command port
```
sudo microcom -D /dev/ttyUSB2
```

```
AT+CGDCONT=1,"IP","three.co.uk"
AT+CGDCONT=2,"IP","three.co.uk"
AT+CGDCONT=3,"IP","three.co.uk"
AT+CGDCONT=4,"IP","three.co.uk"
AT+CGDCONT=5,"IP","three.co.uk"

AT+CGACT: 3,1
```
Exit microcom CTRL ALT x


### Copy the scripts for pppd to use

```
sudo cp pppRC7620 /etc/ppp/peers/
sudo cp chatUp /etc/chatscripts/
sudo cp chatDown /etc/chatscripts/
```

### Make an IP Cellular WAN connections using pppd

```
sudo pppd  /dev/ttyUSB2 115200 call pppRC7620
```

First test log
``` 
Jan 26 20:33:46 raspberrypi pppd[3078]: pppd options in effect:
Jan 26 20:33:46 raspberrypi kernel: [ 9703.506099] PPP generic driver version 2.4.2
Jan 26 20:33:46 raspberrypi pppd[3078]: debug#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: nodetach#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: unit 1#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: dump#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: noauth#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: /dev/ttyUSB2#011#011# (from command line)
Jan 26 20:33:46 raspberrypi pppd[3078]: 115200#011#011# (from command line)
Jan 26 20:33:46 raspberrypi pppd[3078]: nolock#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: connect /usr/sbin/chat -v -f /etc/chatscripts/chatUp#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: disconnect /usr/sbin/chat -v -f /etc/chatscripts/chatDown#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: crtscts#011#011# (from /etc/ppp/options)
Jan 26 20:33:46 raspberrypi pppd[3078]: modem#011#011# (from /etc/ppp/options)
Jan 26 20:33:46 raspberrypi pppd[3078]: noaccomp#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: asyncmap 0#011#011# (from /etc/ppp/options)
Jan 26 20:33:46 raspberrypi pppd[3078]: nomagic#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: nopcomp#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: lcp-echo-failure 4#011#011# (from /etc/ppp/options)
Jan 26 20:33:46 raspberrypi pppd[3078]: lcp-echo-interval 30#011#011# (from /etc/ppp/options)
Jan 26 20:33:46 raspberrypi pppd[3078]: hide-password#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: novj#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: ipcp-accept-local#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: ipcp-accept-remote#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: noipdefault#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: defaultroute#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: replacedefaultroute#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: usepeerdns#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: :#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: noccp#011#011# (from /etc/ppp/peers/pppRC7620)
Jan 26 20:33:46 raspberrypi pppd[3078]: noipx#011#011# (from /etc/ppp/options)
Jan 26 20:33:46 raspberrypi pppd[3078]: pppd 2.4.9 started by pi, uid 0
Jan 26 20:33:47 raspberrypi chat[3086]: abort on (BUSY)
Jan 26 20:33:47 raspberrypi chat[3086]: abort on (NO CARRIER)
Jan 26 20:33:47 raspberrypi chat[3086]: abort on (VOICE)
Jan 26 20:33:47 raspberrypi chat[3086]: abort on (NO DIALTONE)
Jan 26 20:33:47 raspberrypi chat[3086]: abort on (NO DIAL TONE)
Jan 26 20:33:47 raspberrypi chat[3086]: abort on (NO ANSWER)
Jan 26 20:33:47 raspberrypi chat[3086]: abort on (DELAYED^M)
Jan 26 20:33:47 raspberrypi chat[3086]: send (ATZ^M^M)
Jan 26 20:33:47 raspberrypi chat[3086]: expect (OK)
Jan 26 20:33:47 raspberrypi chat[3086]: ATZ^M^M
Jan 26 20:33:47 raspberrypi chat[3086]: OK
Jan 26 20:33:47 raspberrypi chat[3086]:  -- got it
Jan 26 20:33:47 raspberrypi chat[3086]: send (AT^M^M)
Jan 26 20:33:47 raspberrypi chat[3086]: expect (^M)
Jan 26 20:33:47 raspberrypi chat[3086]: ^M
Jan 26 20:33:47 raspberrypi chat[3086]:  -- got it
Jan 26 20:33:47 raspberrypi chat[3086]: send (OK^M)
Jan 26 20:33:47 raspberrypi chat[3086]: expect (AT+CGACT=1,3^M)
Jan 26 20:33:47 raspberrypi chat[3086]:
Jan 26 20:33:47 raspberrypi chat[3086]: AT^M^M
Jan 26 20:33:47 raspberrypi chat[3086]: OK^M
Jan 26 20:34:32 raspberrypi chat[3086]: alarm
Jan 26 20:34:32 raspberrypi chat[3086]: Failed
Jan 26 20:34:32 raspberrypi pppd[3078]: Script /usr/sbin/chat -v -f /etc/chatscripts/chatUp finished (pid 3085), status = 0x3
Jan 26 20:34:32 raspberrypi pppd[3078]: Connect script failed
Jan 26 20:34:33 raspberrypi pppd[3078]: Exit.

````

After this test
```
at+cgact?
+CGACT: 1,1
+CGACT: 2,0
+CGACT: 3,1
+CGACT: 5,0
+CGACT: 4,0
```



