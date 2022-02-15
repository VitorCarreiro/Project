# www

**Instance type:**

**t2.micro**

**IP: 172.31.2.101**

**First config your netplan**

`You can see how to do it here` -> https://github.com/VitorCarreiro/AWS_NAT_IPTABLES_Ubuntu_Redhat

**Update the machine**

`yum update -y`

# Install Apache

`yum install httpd`

**Install ssl for certs**

`yum install mod_ssl`

**Create a soft link**

`ln -s /etc/pki/tls/private private`

**Create your cert and key and place it on here**

`/etc/ssl/certs/www.enta.pt - (For certificate)`

`/etc/ssl/private/www.enta.pt - (For key)`

**Copy the html folder and make the htmls folder for the https page**

`cp -r /var/www/html /var/www/htmls`

`Go to nano /etc/httpd/conf.d/ssl.conf`

**Uncomment and edit for this settings**

`*DocumentRoot "/var/www/htmls"`

`ServerName www.enta.pt:443`

**Comment**

`#SSLProtocol all -SSLv3`

`#SSLCipherSuite HIGH:MEDIUM:!aNULL:!MD5:!SEED:!IDEA`

**Edit your cert and key**

`SSLCertificateFile /etc/ssl/certs/www.enta.pt.crt`

`SSLCertificateKeyFile /etc/ssl/private/www.enta.pt.key`

**systemctl restart httpd**

# ProFTPd Config

**You need to install the epel package to be able to install proftpd** `yum install epel-release`

**Install proftpd**

`yum install proftpd`

**Go to the config file**

`nano /etc/proftpd.conf`

**Create certificates and change to like this**
```
/etc/ssl/certs/ftp.enta.pt.crt
/etc/ssl/private/ftp.enta.pt.key
```
**Add the ports you want to use for your passive ftp, you can place it on the last line**

`PassivePorts 10000 10100`

**Comment these lines**
```
     # <IfDefine TLS>
     #  <IfModule mod_tls_shmcache.c>
     #    TLSSessionCache            shm:/file=/var/run/proftpd/sesscache
     #  </IfModule>
     #</IfDefine>
```
**Change your cert and key to where you placed them**
```
TLSRSACertificateFile         /etc/ssl/certs/ftp.enta.pt.crt
TLSRSACertificateKeyFile      /etc/ssl/private/ftp.enta.pt.key
```
**Create a user if you need to or just use ec2-user**

`adduser client`

**Now that everything is ready start the server**

`systemctl enable --now proftpd`
