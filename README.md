# headless-kali-pi
Confifgure kali 2020.4 as headless on RasPi 4b

### RasPi 4b /boot/config.txt

In order for OS to proceed past bootloader on headless HDMI output needs to be forced. The following settings cause HDMI to output even if no monitor is detected. [See here](https://www.raspberrypi.org/documentation/configuration/config-txt/video.md)

1. Uncomment `hdmi_force_hotplug=1` in `/boot/config.txt`

2. A display resolution also needs to be set:

uncomment following 2 lines in `/boot/config.txt` and set as required (see hdmi_group & hdmi_mode in above link)
```
hdmi_group=<YOUR VALUE>
hdmi_mode=<YOUR VALUE>
```
i.e. for 1080p @ 60hz

```
hdmi_group=1
hdmi_mode=16
```

See [config.txt](../main/config.txt)


### Network
The following configurations ensure that a Wifi connection to your chosen network is established on startup

1. Add entry for wlan0 in `/etc/network/interfaces`

```
auto lo
iface lo inet loopback

auto eth0
allow-hotplug eth0
iface eth0 inet dhcp

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
3. Either create a WiFi connection manually `my.nmconnection` in `/etc/NetworkManager/system-connections`
or in GUI.

4. In GUI once connection is established go to `Advanced Network Connections > [YOUR CONNECTION] > Edit the selected connection (cog icon) > General tab` and make sure `All users may connect to this network` is checked.



### VNC 
The following configuration creates a VNC server that listens on localhost and is activated on startup. Connection to VNC server is established by way of SSH tunnel on client machine.

Install x11vnc:
```
sudo apt update
sudo apt install x11vnc
```

Create a service for VNC:

Copy [x11vnc.service](../main/x11vnc.service) to `/etc/systemd/system/` [Please note that you may need to adjust flags depending on your requirements]

Start service: `sudo systemctl start xllvnc.service`
Check status: `sudo systemctl status x11vnc.service`

If all goes well you should receive the following status message:

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
Enable VNC on boot: `sudo systemctl enable x11vnc.service`






`ssh -i ~/.ssh/<YOUR KEY> -L 5900:localhost:5900 -N f kali@<YOUR IP>`
