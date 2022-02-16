# Marketing

**t2.micro**

**IP: 172.31.1.104**

**Do this to enable net**

**Go to nano /etc/netplan/50-cloud-init.yaml and add the gateway of your server**
```
            dhcp4-overrides:
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
**Install RDP**

(On root)

`sudo apt install -y xfce4 xfce4-goodies`

`sudo apt install -y xrdp chromium-browser filezilla thunderbird`

`sudo adduser xrdp ssl-cert`

(Leave root)

`echo xfce4-session > ~/.xsession`

`sudo reboot`

`sudo passwd ubuntu`

**Install NFS client

`mkdir /mnt/raid5_contas`

**Mount to the nfs server**

`mount -t nfs 172.31.1.101:/mnt/raid5_contas /mnt/raid5_contas`

**Do `cat /etc/mtab` and copy the last line**

**Do `nano /etc/fstab` and paste it there with nofail**

# NIS Client

**Go to nano `/etc/yp.conf` and add**

`domain inova.pt server central.inova.pt`

**Now go to nano /etc/nsswitch.conf and add nis on those lines**
```
passwd:         files systemd nis
group:          files systemd nis
shadow:         files nis
hosts:          files dns nis
```
**Go to `nano /etc/pam.d/common-session` and add this at the end**
`session optional pam_mkhomedir.so skel=/etc/skel umask=077`

**Restart nis and your good to go**

`systemctl restart nis`

**If you want to login to the user**

`login user`
