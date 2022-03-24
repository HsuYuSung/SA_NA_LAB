
# LAB-4


## Step
```bash
# 1. Create two disk in virtual machine
# 2. Partitioning
	- `sudo gdisk /dev/sdb`
		1. `o`: create a new empty GUID partition table (GPT)
		2. `p`: print the partition table
		3. `n`: add a new partition
		4. `p`: write table to disk and exit

# 3. Create raid0
sudo mdadm --create --auto=yes --level=0 --raid-devices=2 /dev/md0 --metadata=1.2 /dev/sdb1 /dev/sdc1

# 4. Setup filesystem
sudo mkfs.ext4 /dev/md0

# 5. mount filesystem
sudo mount /dev/md0 /mnt

# 6. Write config into /etc/mdadm/mdadm.conf
sudo mdadm --detail --scan --verbose | sudo tee -a /etc/mdadm/mdadm.conf

# 7.
sudo update-initramfs -u
```
