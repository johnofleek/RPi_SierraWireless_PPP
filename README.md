# A collection of ppp and chat scripts

## Purpose
Enable the raspberry pi to make a ppp connection to the internet via the Pilot board fitted with 
Sierra Wireless HLxxxx modem

## Tested so far on

| OS | Modem(s) |
| --- | --- |
| Jessie | HL7698 HL8548 |
| Stretch | HL7698 |




Note that the reason pppd's built in authentication isn't used is that some of the modules don't like doing that

# Get started - RPi command line

## Copy the scripts for pppd to use
```
sudo apt-get install ppp
sudo cp pppPilot /etc/ppp/peers/
sudo cp chatUp /etc/chatscripts/
sudo cp chatDown /etc/chatscripts/
```
## Configure the HL module with the SIM card credentials
The chat script is written to automate configuration of modem APN, PAP or CHAP authentication settings. Alternatively this can be carried out manually using a terminal and AT commands.

Notes 
-    The example uses PAP autentication
-    If using a physical serial port - baud rate and handshake settings will be needed. 
-    If the sim doesn't require authentication credential settings - set both the username and password to empty strings ""
-    This script only needs to be run when the modem settings need to be changed - the modem retains the settings after a power cycle

Edit the file "chatHLsetup" to match the credentials required for your SIM card. 

Then execute the chat script as follows 
```
chat -v -f ./chatHLsetup > /dev/ttyACM0 < /dev/ttyACM0
```

## Make an IP connection with the Pilot / HL
Note that serial port ttyACM0 is used 
	- ttyACM2 and the hardware serial port ttyAMA0 have not at this time been tested

sudo pppd  /dev/ttyACM0 115200 call pppPilot


## Debugging
### Check script execution - do this in another shell terminal
```
tail -f /var/log/syslog
```

### Talk to the modem directly from the raspberry pi
For details on the modem AT command manual at source.sierrawireless.com
```
sudo apt-get install minicom
sudo minicom -D /dev/ttyACM0
```

### Check the IP interface with **Example response**
````
ifconfig ppp1

ppp1: flags=4305<UP,POINTOPOINT,RUNNING,NOARP,MULTICAST>  mtu 1500
        inet 172.26.168.84  netmask 255.255.255.255  destination 172.26.168.84
        ppp  txqueuelen 3  (Point-to-Point Protocol)
        RX packets 4  bytes 58 (58.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 4  bytes 64 (64.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

```
