A collection of ppp and chat scripts

Enables the raspberry pi to make a ppp connection to the internet. 
Using ppp. Via a Sierra Wireless HL module

Tested so far on
Jessie	HL7698	HL848
Stretch HL7698

The chat script is written to allow PAP or CHAP authentication
Note that the reason pppd's built in authentication isn't used is that the modules don't like doing that

# Get started - RPi command line

sudo apt-get install ppp
sudo cp streamWpppACM0 /etc/ppp/peers/
sudo cp streamWppp /etc/chatscripts/
sudo cp streamWpppdDisconnect /etc/chatscripts/

## make the connection
sudo pppd call streamWpppACM0

# debug
## Check script execution
tail -f /var/log/syslog

## Talk to the modem 

sudo apt-get install minicom
sudo minicom -D /dev/ttyACM0
