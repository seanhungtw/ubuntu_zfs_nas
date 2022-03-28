# ubuntu_zfs_nas
a note to build a nas on ubuntu

2022/03/28

# working on Ubuntu 21.10
# ========== ZFS command ==========

# install zfs:
sudo apt install zfsutils-linux

# import exist zfs pool
  zpool import

  zpool status

  zpool scrub raid1
  zpool list
  lsblk -f
  zfs get all

# create a zpool
  # mirror ( equal to raid1)
  sudo zpool create -f raid1 mirror /dev/sdc /dev/sdd
  # set compression (optional)
  sudo zfs set compression=zstd raid1
  # set mouint point
  sudo zfs set mountpoint=/mnt/raid1 raid1
  sudo zfs mount raid1

# Useful command:
  # check compresstion ratio
  zfs get compressratio raid1
  
  # check hdd & file system (you should do it often weekly or mounthly)
  sudo zfs get dedup pool1
  
  # disable dedup ( default is off, actually )
  zfs set dedup=off pool1
  
  # check zpool status
  zpool iostat -v
  zpool status -v raid1

# Delete a zfs pool (be careful!)
  # zpool destory raid1

# ========== Samba configuration ==========

# install samba
  sudo apt install samba
# create samba user
  sudo useradd sambauser
  sudo passwd sambauser
  sudo smbpasswd -a sambauser

# edit /etc/samba/smb.conf
  [global]

  hosts allow = 10.0.0.0/24
  hosts deny = ALL

  # for symblolic links ( optional )
  follow symlinks = yes
  wide links = yes
  unix extensions = no
  # security setting, SMB1 is not secure
  min protocol = SMB3
  # set user
  [sambauser]
  path = /mnt/raid1/
  available = yes
  valid users = sambauser
  read only = no
  browseable = yes
  #public = yes
  writable = yes
  
# ========== static ip ==========
sudo vi /etc/netplan/00-installer-config.yaml


# edit /etc/netplan/00-installer-config.yaml
  # This is the network config written by 'subiquity'
  network:
    ethernets:
      enp3s0:
        #      dhcp4: true
        addresses: [ 10.0.0.1/24 ]
    version: 2

  sudo netplan try
  sudo netplan apply

# ========== useful tools ==========
  sudo apt-get install lm-sensors
  sudo apt-get install iperf3
  
  #===========get uuid
  # ls -l /dev/disk/by-uuid/
  total 0
  lrwxrwxrwx 1 root root 10 Jan 25 07:23 15894965499706790219 -> ../../sdd1
  lrwxrwxrwx 1 root root 10 Jan 25 07:23 7ed8c0d1-ee3a-4ba4-aa10-95e731dadc0c -> ../../sde1
  lrwxrwxrwx 1 root root 10 Jan 25 07:23 80a3448c-f771-423c-9ff5-19594ef2fa78 -> ../../sdf1
  lrwxrwxrwx 1 root root 10 Jan 25 07:23 84a14034-5ad4-4d90-9294-74ce7ab84df5 -> ../../sdb1
  lrwxrwxrwx 1 root root 10 Jan 25 07:23 9078-0388 -> ../../sda1
  lrwxrwxrwx 1 root root 10 Jan 25 07:23 969eab0f-6bb1-11e8-acf7-7085c27556f6 -> ../../sda2

  #==========auto mount
  # edit /etc/fstab
  UUID=84a14034-5ad4-4d90-9294-74ce7ab84df5 /mnt/hdd1 ext4 defaults  0       2
  UUID=7ed8c0d1-ee3a-4ba4-aa10-95e731dadc0c /mnt/hdd2 ext4 defaults  0       2
  UUID=80a3448c-f771-423c-9ff5-19594ef2fa78 /mnt/hdd3 ext4 defaults  0       2
