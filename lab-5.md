
# LAB-05

## Step

```bash

# 1. compile pi.c

# 2. move pi
mv pi to /opt/

# 3. move pi.service
mv pi.service to /etc/systemd/system/

# 4. edit pi.service

# 5. start daemon
sudo systemctl daemon-reload
sudo systemctl start pi
sudo systemctl restart pi
```
### file
```bash
# /etc/systemd/system/pi.service
[Unit]
Description=pi service

[Service]
MemoryHigh=10M
CPUQuota=10%
Type=simple
ExecStart=/opt/pi
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
# /etc/crontab
* * * * * root  date +\%s | tee /var/log/pi.log
```