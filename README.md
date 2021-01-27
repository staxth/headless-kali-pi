# headless-kali-pi
Confifgure kali 2020.4 as headless on RasPi 4b

#### RasPi 4b /boot/config.txt
[config.txt](../main/config.txt)

#### VNC 
Install x11vnc:
```
sudo apt update
sudo apt install x11vnc
```

Create a service for vnc:

`sudo nano /etc/systemd/system/x11vnc.service`

Paste contents of [x11vnc.service](../main/x11vnc.service) into editor

Start service `sudo systemctl start xllvnc.service`
Check status `sudo systemctl status x11vnc.service`

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






`ssh -i ~/.ssh/<YOUR KEY> -L 5900:localhost:5900 -N f kali@<YOUR IP>`
