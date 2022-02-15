# Wazuh

**Instance type:**

**t2.medium**

**IP: 172.31.1.102**

**Do this to enable net**

**Go to `nano /etc/netplan/50-cloud-init.yaml` and add the gateway of your server**
```
            dhcp4-overrides:
                use-dns: false
                use-routes: false
            routes:
              - to: 0.0.0.0/0
                via: 172.31.1.100
                on-link: true
            nameservers:
                search: [inova.pt]
                addresses: [172.31.0.100]
```
**Try and save**
```
netplan try
netplan apply
```
**Update and upgrade**

`apt update && apt upgrade -y`

## Wazuh

To install wazuh you should go to this page since every step to follow is there and it works perfectly

➡️ https://documentation.wazuh.com/current/installation-guide/open-distro/all-in-one-deployment/all-in-one.html

## RAID6

**Create the partition for your disks with gdisk**
```
gdisk /dev/xvdf
gdisk /dev/xvdg
gdisk /dev/xvdh
gdisk /dev/xvdi
```
**Now create a RAID 6 array**

`mdadm --create /dev/md2 --level 6 --raid-devices 4 /dev/xvdf1 /dev/xvdg1 /dev/xvdh1 /dev/xvdi1`

**Check if everything is alright and wait for the State: Clean**

`mdadm --detail /dev/md2`

**Now encrypt your folder with LUKS**

`cryptsetup luksFormat --hash=sha512 --key-size=512 --cipher=aes-xts-plain64 --verify-passphrase /dev/md2`

**Use this command to initialize the volume and create the mapping**

`cryptsetup luksOpen /dev/md2 md2_crypt`

**Create a physical volume with LVM**

`vcreate /dev/mapper/md2_crypt`

**Create a volume group for your physical**

`vgcreate vg2 /dev/mapper/md2_crypt`

**Create a logical volume of your volume**

`lvcreate -n vg2lv2 -l +100%FREE vg2`

**Construct a filesystem, I used ext4**

`mkfs.ext4 /dev/vg0/vg2lv2`

**Create a folder at /mnt/**

`mkdir /mnt/raid5_data`

**Now mount the disk**

`mount /dev/vg2/vg2lv2 /mnt/raid5_data/`

**Do cat `/etc/mtab` and copy the last line "/dev/mapper/vg2....."**

**Now go to `nano /etc/fstab` and place nofail there like this**

![image](https://user-images.githubusercontent.com/98783977/154141081-cfe0dfca-7697-48d5-b96e-77019e882004.png)

**"Optional" - If you want to add some extra keys to your folder do**

`cryptsetup luksAddKey /dev/md2`

**Everytime you reboot you have to do this**

**`cryptsetup luksOpen /dev/md2 md2_crypt` and then `mount -a`**

