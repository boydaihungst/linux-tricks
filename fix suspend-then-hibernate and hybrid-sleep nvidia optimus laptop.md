 Fix hybrid-sleep, suspend-then-hibernate for laptop with nvidia Optimus (Nvidia + intel, or Nvidia + amd) 


1. Created two systemd units:

`sudo nano /usr/lib/systemd/system/nvidia-suspend-then-hibernate-hybrid-sleep.service`

```systemd
[Unit]
Description=NVIDIA system suspend-then-hibernate and hybrid-sleep actions
Before=systemd-suspend-then-hibernate.service
Before=systemd-hybrid-sleep.service

[Service]
Type=oneshot
ExecStart=/usr/bin/logger -t suspend -s "nvidia-suspend-then-hibernate-hybrid-sleep.service"
ExecStart=/usr/bin/nvidia-sleep.sh "hibernate"

[Install]
WantedBy=systemd-suspend-then-hibernate.service
WantedBy=systemd-hybrid-sleep.service
```

2. Edit unit `/usr/lib/systemd/system/nvidia-resume.service`
```sh
sudo systemctl edit --full nvidia-resume.service
```
```systemd
[Unit]
Description=NVIDIA system resume actions
After=systemd-suspend.service
After=systemd-hibernate.service
After=systemd-suspend-then-hibernate.service
After=systemd-hybrid-sleep.service

[Service]
Type=oneshot
ExecStart=/usr/bin/logger -t suspend -s "nvidia-resume.service"
ExecStart=/usr/bin/nvidia-sleep.sh "resume"

[Install]
WantedBy=systemd-suspend.service
WantedBy=systemd-hibernate.service
WantedBy=systemd-suspend-then-hibernate.service
WantedBy=systemd-hybrid-sleep.service
```

3. Remove file `/usr/lib/systemd/system-sleep/nvidia`
```sh
sudo rm -rf /usr/lib/systemd/system-sleep/nvidia
```
4. Enable services

```sh
sudo systemctl daemon-reload
sudo systemctl disable nvidia-resume.service
sudo systemctl enable nvidia-suspend.service
sudo systemctl enable nvidia-hibernate.service
sudo systemctl enable nvidia-suspend-then-hibernate-hybrid-sleep.service
sudo systemctl enable nvidia-resume.service
```

4. Test with commands:
```sh
systemctl suspend-then-hibernate
systemctl hybrid-sleep
```

5. Optional
`/etc/systemd/logind.conf`

```config
LidSwitchIgnoreInhibited=yes
HandleLidSwitch=hybrid-sleep
HandleLidSwitchExternalPower=hybrid-sleep
HandleLidSwitchDocked=hybrid-sleep
IleAction=hybrid-sleep
IdleActionSec=1800 # Go to hybrid-sleep mode after idle for 30m
```
