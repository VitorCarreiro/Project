# www

**Instance type:
t2.micro
IP: 172.31.2.101**

**First config your netplan**

`You can see how to do it here` -> https://github.com/VitorCarreiro/AWS_NAT_IPTABLES_Ubuntu_Redhat

**Update the machine**

`yum update -y`

**Install Apache**

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

