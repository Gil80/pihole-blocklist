Follow this guide
https://www.youtube.com/watch?v=Kb6v1GqeAc0
Or the text version: https://medium.com/@Arafat./graphical-user-interface-using-vnc-with-amazon-ec2-instances-549d9c0969c5
I used this article to troubleshoot grey desktop: https://forums.fast.ai/t/vnc-server-rfb-003-008/14607/19 by editing `~/.vnc/xstartup`

Replace anything in the `xstartup` with this:
```xsetroot -solid grey
vncconfig -iconic &
x-terminal-emulator -geometry 80x24+10+10 -ls -title "$VNCDESKTOP Desktop" &
x-window-manager &

gnome-panel &
gnome-settings-daemon &
metacity &
nautilus &
lxterminal &
/usr/bin/lxsession -s LXDE &
```

run command: `vncserver :1 -geometry 1024x768`

This is an example of tunnling to my oracle instance after setting up VNC: `ssh -L 5902:localhost:5902 -i ~/.ssh/ssh-key-2021-07-27.key ubuntu@140.238.203.129`

Download VNC viewer and in the address set: `localhost:5902`


Alternatives:
https://www.floydimus.prismo.net/Floydimus/hostsv2
