# Central

**Instance type:**

**t2.micro**

**172.31.1.101**

**Do this to enable net**

Go to nano /etc/netplan/50-cloud-init.yaml and add the gateway of your server
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
**Update and upgrade**
`apt update && apt upgrade -y`

# NFS Server

**Install nfs server**

`apt install nfs-kernel-server`

**Go to `nano /etc/exports` and place something like this(syncronized with raid) **

`/mnt/raid5_contas *(rw,sync,no_subtree_check,no_root_squash)`

**Do this to export or unexport all directories**

`exportfs -a`

**Start and enable server**

`systemctl enable --now nfs-kernel-server`

**Add a user**

`adduser vin --home /mnt/raid5_contas/vin`

**And your done with NFS server**

# NIS Server

**Install nis**

apt -y install nis

**Go to nano /etc/default/nis and change to this**
```
NISSERVER=master

NISCLIENT=false
```
**Go to nano /etc/ypserv.securenets and place your netmask and IP**
```
#0.0.0.0 0.0.0.0

255.255.0.0 172.31.0.0
```
**Go to /var/yp/Makefile and change from false to true**
```
MERGE_PASSWD=true

MERGE_GROUP=true
```
**Do your /etc/hosts**
```
172.31.1.101 central.inova.pt central inova.pt
172.31.0.100 control.inova.pt control inova.pt
```
**Update your database of nis with**

`/usr/lib/yp/ypinit -m`

**Go to `cd /var/yp`**

**And do `make`**

`systemctl restart nis`

# Email Server

**Install these packages**

`apt install postfix postfix-doc dovecot-pop3d dovecot-imapd sasl2-bin`

**Go to nano /etc/default/saslauthd ,change START= YES and change OPTIONS to this one**

![image](https://user-images.githubusercontent.com/98783977/154155060-6ec433ba-548c-4493-8a8a-7b194b730598.png)

![image](https://user-images.githubusercontent.com/98783977/154155172-4f21641c-3f1f-469f-b79c-bff8597963d3.png)

**Start services**
```
systemctl start saslauthd
systemctl start postfix
```
**Go to nano /etc/dovecot/conf.d/10-mail.conf and change to this**

![image](https://user-images.githubusercontent.com/98783977/154155438-00917ca2-10ee-4273-9d08-5f09cc0ac49a.png)

**Go to nano /etc/dovecot/conf.d/10-auth.conf and remove # from disable**

![image](https://user-images.githubusercontent.com/98783977/154155638-32afd5a2-0a93-4dbf-a7e6-b31374f2ae85.png)

**Start and enable dovecot**
```
systemctl start dovecot
systemctl enable dovecot
systemctl restart dovecot
```
**Go to cd /etc/skel/ and type this**

`maildirmake.dovecot Maildir`

**Go to cd /etc/skel/Maildir/ and create users**
```
adduser lemos
adduser joao
adduser vin
```
Go to cd /home and type this

`dpkg-reconfigure postfix`

cd ubuntu/

`maildirmake.dovecot Maildir`

cd /etc/postfix/

