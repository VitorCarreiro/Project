# Sales

**Instance type:**

**t2.micro**

**IP: 172.31.1.103**

**First config your netplan**

**You can see how to do it here**-> https://github.com/VitorCarreiro/AWS_NAT_IPTABLES_Ubuntu_Redhat

**Update the machine**

`yum update -y`

# RDP Config

**Install the epel package**

`sudo amazon-linux-extras install epel`

**Install MATE desktop**

`yum groupinstall -y "MATE Desktop"`

**Install xrdp**

`yum install -y xrdp`

**Install these files**

`yum install chromium filezilla thunderbird`

**Change your password of ec2-user**

`passwd ec2-user`

**Start xrdp**

`systemctl start xrdp`

**Do this so that all your users can access the RDP**

`sudo bash -c 'echo PREFERRED=/usr/bin/mate-session > /etc/sysconfig/desktop'`

# NFS Client

**Install NFS**

`yum install nfs-utils`

**Create the folder you made on the server**

`mkdir /mnt/nfs`

**Mount the system**

`mount -t nfs 172.31.1.101:/mnt/nfs/ /mnt/nfs/`

**Do `cat /etc/mtab` and copy the last line**

Now go to `nano /etc/fstab` and paste the line there with ,nofail, between some words

**And that is it, you should now be able to enter the folder and create files!**

