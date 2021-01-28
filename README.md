# headless-kali-pi
Confifgure kali 2020.4 as headless on RasPi 4b

#### RasPi 4b /boot/config.txt

In order for OS to proceed past bootloader on headless HDMI output needs to be forced. The following settings cause HDMI to output even if no monitor is detected. [See here](https://www.raspberrypi.org/documentation/configuration/config-txt/video.md)

1. uncomment `hdmi_force_hotplug=1`in `/boot/config.txt`

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

#### VNC 
Install x11vnc:
```
sudo apt update
sudo apt install x11vnc
```

Create a service for vnc:

Copy [x11vnc.service](../main/x11vnc.service) to `/etc/systemd/system/`

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
