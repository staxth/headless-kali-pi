# headless-kali-pi
Confifgure kali 2020.4 as headless on RPi4b

This configuration was tested and works with:
- Raspberry Pi Model 4B 4GB
- [Kali Linux RaspberryPi 2 (v1.2), 3, 4 and 400 (64-Bit) (img.xz)](https://www.offensive-security.com/kali-linux-arm-images/)

This configuration results in a RPi4 running headless (or headed) kali linux with a secure VNC service enabled at startup. 

### Contents
 1. Installation
 2. Requirements
 3. RPi4b /boot/config.txt
 4. Network
 5. VNC
 6. Autologin (optional)
 7. Setup Script (TBC)


### 1. Installation 
[Kali Linux on RPi](https://www.kali.org/docs/arm/kali-linux-raspberry-pi/)

I used [Raspberry Pi Imager](https://www.raspberrypi.org/software/) to image Kali to the sd card.


### 2. Requirements

As it is not possible to `ssh` into the RPi until steps 3 & 4 are complete you will need a way to modify the sd card contents.

If using the RPi in a headed scenario then all configurations can be made while running. Alternativly configurations can be made 
by mounting both the `/boot` and `/root` paritions of the sd card on a linux system. `/boot/config.txt` is located in `/boot` and the Kali root
file system in `/root`.


### 3. RPi4b /boot/config.txt

In order for OS to proceed past bootloader and load into desktop on headless RPi4 the device must be forced to output HDMI. 

see thread [here](https://www.raspberrypi.org/forums/viewtopic.php?t=253312)

> The reason is that by default (RPi4), if no screen is connected at boot then a display device is not created. Without a 
> display device the GUI desktop does not start so any program that requires GUI will not start. Other RPi models did not have that issue 
> because they would fall back to composite mode if no HDMI was connected.
>
> klricks

The following setting forces HDMI to output even if no monitor is detected. [Video options in config.txt](https://www.raspberrypi.org/documentation/configuration/config-txt/video.md)

1. Uncomment `hdmi_force_hotplug=1` in `/boot/config.txt`

2. A display resolution also needs to be set:

     Uncomment following 2 lines in `/boot/config.txt` and set as required (see hdmi_group & hdmi_mode in above link)
     ```
     hdmi_group=<YOUR VALUE>
     hdmi_mode=<YOUR VALUE>
     ```
     i.e. for 1080p @ 60hz
     ```
     hdmi_group=1
     hdmi_mode=16
     ```

     See [config.txt](../main/config.txt) in repo for working example.

Once these changes have been made the RPi4 will boot to login screen in both headless and headed scenarios.


### 4. Network
The following configurations ensure that a Wifi connection to your chosen network is established on startup.

1. Add entry for wlan0 in `/etc/network/interfaces`

     ```
     auto lo
     iface lo inet loopback

     auto eth0
     allow-hotplug eth0
     iface eth0 inet dhcp
     
     # Added lines
     auto wlan0 
     iface wlan0 inet dhcp 
     ```
2. Set `[ifupdown]` in `/etc/NetworkManager/NetworkManager.conf` to `managed=true`
     ```
     [main]
     plugins=ifupdown,keyfile

     [ifupdown]
     managed=true
     ```

3. Either create a WiFi connection manually: `<YOUR CONNECTION>.nmconnection` in `/etc/NetworkManager/system-connections`
or via GUI.

4. In both cases once a connection is established use GUI (headed )and go to `Advanced Network Connections > [YOUR CONNECTION] > Edit the selected connection (cog icon) > General tab` and make sure `All users may connect to this network` is checked.


### 5. VNC 
The following configuration creates a VNC server that listens on localhost and is activated on startup. Connection to the VNC server is established by way of SSH tunnel on client machine.

Install x11vnc:
```
sudo apt update
sudo apt install x11vnc
```
1. Create a service for VNC:

Copy [x11vnc.service](../main/x11vnc.service) to `/etc/systemd/system/` [Please note that you may need to adjust flags depending on your requirements] A password for the service can be created with `vncpasswd` 

2. Start service: `sudo systemctl start xllvnc.service`

3. Check status: `sudo systemctl status x11vnc.service`

If the service starts succesfully `systemctl` will display the following:
```
● x11vnc.service - start X11VNC at startup
     Loaded: loaded (/etc/systemd/system/x11vnc.service; enabled; vendor preset: disabled)
     Active: active (running) since Tue 2021-01-26 19:12:58 EET; 1 day 4h ago
   Main PID: 737 (x11vnc)
      Tasks: 1 (limit: 4490)
        CPU: 2min 304ms
     CGroup: /system.slice/x11vnc.service
             └─737 /usr/bin/x11vnc -display :0 -auth guess -rfbauth /home/kali/.vnc/passwd -rfbport 5900 -listen 127.0.0.1 -xkb -quiet -forever -o /var/log/x11>
```
4. Enable VNC on boot: `sudo systemctl enable x11vnc.service`

To connect to service install a viewer on client machine and set up a SSH tunnel:

`ssh (-i ~/.ssh/<YOUR KEY>) -L 5900:localhost:5900 -N f kali@<YOUR IP>`

You can then access the remote desktop by opening a connection to localhost on the client machine i.e. `open vnc://localhost:5900` depending on your flag settings you may need to provide the vnc password that you created earlier.


### 6. Autologin (optional)
This is neither required nor secure, but if you so desire autologin can be enabled by modifying
`/etc/lightdm/lightdm.conf` and changing the following line in Seat
```
[Seat:*]
...
#autologin-user= << uncomment and change to autologin-user=kali
#autologin-user-timeout=0
#autologin-in-background=false
#autologin-session=
#exit-on-failure=false
```
No other changes should be required.

### Test

Once the above has been configured you should have a working RPi4 headless kali build with VNC and network support on startup. 
Further changes / steps will be added as needed.






