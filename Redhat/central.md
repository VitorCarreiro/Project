# Central

**Instance type:**

**t2.micro**

**172.31.1.101**

**First config your netplan so you can have net**

`You can see how to do it here` -> https://github.com/VitorCarreiro/AWS_NAT_IPTABLES_Ubuntu_Redhat

**Update the machine**

`yum update -y`

# Install NFS Server

`yum install nfs-utils`

**Start and enable NFS**

`systemctl start nfs-server.service and systemctl enable nfs-server.service`

**Create folder that you'd like to export to clients**

`mkdir -p /mnt/nfs`

**Configure the /etc/exports file to be something like this**

`/mnt/nfs *(rw,sync,no_subtree_check,no_root_squash)`

**Use this command to export or unexport all directories**

`exportfs -a`

**Add a user to the nfs folder**

`adduser johan --home /mnt/nfs/johan`

