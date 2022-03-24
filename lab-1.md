# LAB-01



- Requirements
	- Make sure VM connected to the internet before judgoing.
- Setup environment
	- `sudo apt install openssh-server wireguard resolvconf`
- Add username with student id
	- `sudo adduser [STUDENT_ID]`
- Add sudo permision without password to the user
	- `sudo bash -c 'echo "[STUDENT_ID] ALL=(ALL:ALL) NOPASSWD:ALL" > /etc/sudoers.d/[STUDENT_ID]'`
- Test SSH server
	- vm
		- `ip a # lookup vm ip address`
	- host computer
		- `ssh [STUDENT_ID]@[vm ip]`
		- e.g. `ssh f74104749@192.168.0.107`
	- import wg config file
		-  Copy wg config file to VM
		- `scp [FILE] [USER@IP]:[LOCATE]`
	- Move wg config to wireguard settings
		- `sudo mv wan.conf /etc/wireguard`
	- Start the wireguard VPN services
		- `sudo wg-quick up wan`
		- `systemctl start wg-quick@wan.service`
	- Auto start wireguard VPN services at boot
		- `sudo systemctl enable wg-quick@wan.service`
	- VPN check
		- 請用 `ip a` 確認你的 vpn 網卡是否啟動  若有請用 `wg show` 確認 `transfer: ? received, XX sent`  中的問號是否為 0  若為 0 請重啟 vpn  重啟後過1分鐘若依然為0請更改您所使用的網路環境試試看。
	- Import ssh authorized_keys

