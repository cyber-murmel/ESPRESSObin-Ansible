# ESPRESSObin-Ansible

based on [archlinux ARM entry](https://archlinuxarm.org/platforms/armv8/marvell/espressobin)


## Preperation

These are the Steps you can take to get a running ESPRESSObin with [Arch Linux](https://archlinuxarm.org/platforms/armv8/marvell/espressobin).


### Make bootable SD card

My SD card is `/dev/mmcblk0`.
```bash
### Zero the beginning of the SD card
sudo dd if=/dev/zero of=/dev/mmcblk0 bs=1M count=8 status=progress
### Partition the SD card
sudo partprobe; echo 'o\np\nn\np\n1\n\n\np\nw\n' | sudo fdisk /dev/mmcblk0; sudo partprobe
### Create the ext4 filesystem
sudo mkfs.ext4 -O '^metadata_csum,^64bit' /dev/mmcblk0p1
### Mount the filesystem
mkdir -f root
sudo mount /dev/mmcblk0p1 root
### Download and extract the root filesystem
wget http://os.archlinuxarm.org/os/ArchLinuxARM-espressobin-latest.tar.gz
sudo su -c 'bsdtar -xpf ArchLinuxARM-espressobin-latest.tar.gz -C root'
### Unmount the partition
sudo umount root
sync
```


### U-Boot configuration

Plug SD card into ESPRESSObin, connect ESPRESSObin to your machine via Micro USB, connect to LAN via Ethernet port nexto USB3.0 and supply 12VDC.

```bash
picocom /dev/ttyUSB0 -b 115200
[...]
Marvell>> mmc dev 0
ext4load mmc 0 $loadaddr /boot/uEnv.txt
env import -t $loadaddr $filesize
saveenv
boot
```
Login with `alarm`:`alarm`
```bash
### Get IP address
ip --brief --color address list dev wan
exit
```
Exit picocom with `Ctrl+a+c`.

### Connect via SSH

To have the SSH server in you `known_hosts` file connect per SSH.

```bash
ssh alarm@10.42.23.143
# You don't need to log in.
# Exit with Ctrl+c
```


## Running Ansible

Change the `ansible_host` in the `inventories/hosts` file to the IP address of your device.


### Bootstrapping

The first part is to run the `bootstrap.yml` This file is quite dirty code (Ansible's [raw module](https://docs.ansible.com/ansible/2.6/modules/raw_module.html)). It needs onlye to be run once. The purpose is to upgrade the machine, install essential packages, give sudo rights to the `alarm` user and copy over your ssh key.

It expects you public ssh key in `~/.ssh/id_rsa.pub`.

```bash
ansible-playbook bootsrap.yml
```

Once this is done, you should be able to login to the machine without a password prompt. `ssh alarm@192.168.188.107`


### Running the Main Playbook

```bash
ansible-playbook site.yml
```
