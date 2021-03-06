# Wazuh 

**Instance type:**

**t2.medium**

**IP: 172.31.1.102**

**First config your netplan**

**You can see how to do it here** -> https://github.com/VitorCarreiro/AWS_NAT_IPTABLES_Ubuntu_Redhat

**Update the machine**

`yum update -y`

**To install wazuh you should go to this page since every step to follow is there**

-> https://documentation.wazuh.com/current/installation-guide/open-distro/all-in-one-deployment/all-in-one.html

**Create the partition for your disks with gdisk**
```
gdisk /dev/xvdf
gdisk /dev/xvdg
gdisk /dev/xvdh
gdisk /dev/xvdi
```

**Now create a RAID 6 array**

`mdadm --create /dev/md0 --level 6 --raid-devices 4 /dev/xvdf1 /dev/xvdg1 /dev/xvdh1 /dev/xvdi1`

**Check if everything is alright and wait for the State: Clean**

`mdadm --detail /dev/md0`

**Now encrypt your folder with LUKS**

`cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/md0`

**Use this command to initialize the volume and create the mapping**

`cryptsetup luksOpen /dev/md0 md0_crypt`

**Create a physical volume with LVM**

`vcreate /dev/mapper/md0_crypt`

**Create a volume group for your physical**

`vgcreate vg0 /dev/mapper/md0_crypt`

**Create a logical volume of your volume**

`lvcreate -n vg0lv0 -l +100%FREE vg0`

**Construct a filesystem, I used XFS**

`mkfs.xfs /dev/vg0/vg0lv0`

**Create a folder at /mnt/**

`mkdir /mnt/raid5_homes`

**Now mount the disk**

`mount /dev/vg0/vg0lv0 /mnt/raid5_homes/`

**Do `cat /etc/mtab` and copy the last line "/dev/mapper/vg0....."**

**Now go to `nano /etc/fstab` and place,nofail, there**

**Something like this ",attr2,nofail,inode64,"**

**"Optional" - If you want to add some extra keys to your folder do**

`cryptsetup luksAddKey /dev/md0`

**Everytime you reboot you have to do this**

`cryptsetup luksOpen /dev/md0 md0_crypt` and then `mount -a`
