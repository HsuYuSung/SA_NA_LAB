# Lab-07



## Requirements

 
將 VM 的 Host name 設為 switch 並 在 VM 上新增兩 個 interfaces 將其依序命名
為 "lan1" "lan2"  將  兩 個 interfaces bridge 起來並將 bridge interface name 設為
LAN 並參 附  A 設定 IP address


## Step


* command
```bash=
cd /etc/netplan
sudo vim 00-installer-config.yaml
sudo netplan apply

```


* 00-installer-config.yaml
```bash=
# This is the network config written by 'subiquity'
network:
  ethernets:
    eth:
      match:
        macaddress: "" # macaddress
      set-name: eth
      dhcp4: true
    lan1:
      match:
        macaddress: "" # macaddress
      set-name: lan1
      dhcp4: true
    lan2:
      match:
        macaddress: "" # macaddress
      set-name: lan2
      dhcp4: true
  bridges:
    LAN:
      interfaces: [ lan1, lan2 ]
      addresses: [ 192.168.3.254/24 ]
  version: 2
```


