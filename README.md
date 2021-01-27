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

`nano /etc/systemd/system/x11vnc.service`

Paste contents [x11vnc.service](../main/x11vnc.service)

Start service `sudo systemctl start xllvnc.service`









`ssh -i ~/.ssh/<YOUR KEY> -L 5900:localhost:5900 -N f kali@<YOUR IP>`
