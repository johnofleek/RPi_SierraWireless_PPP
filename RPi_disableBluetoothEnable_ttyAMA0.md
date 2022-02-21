Using raspi-config  Enable the serial port but don't enable the shell


Add device tree to /boot/config.txt to disable the bluetooth module.
```
sudo nano /boot/config.txt
```

Add at the end of the file for pi4
```
dtoverlay = disable-bt
```

then
```
sudo reboot
```

Check it
```
ls -l /dev/serial*
lrwxrwxrwx 1 root root  7 Feb 16 16:22 /dev/serial0 -> ttyAMA0
lrwxrwxrwx 1 root root  5 Feb 16 16:22 /dev/serial1 -> ttyS0

/dev/serial:
total 0
drwxr-xr-x 2 root root 100 Feb 16 16:22 by-id
drwxr-xr-x 2 root root 100 Feb 16 16:22 by-path
pi@raspberrypi:~ $
```