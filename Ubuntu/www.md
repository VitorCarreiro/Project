# www

**Do this to enable net**

**Go to nano /etc/netplan/50-cloud-init.yaml and add the gateway of your server**
```
            dhcp4-overrides:
                use-routes: false
            routes:
              - to: 0.0.0.0/0
                via: 172.31.2.100
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

# Nginx

**Install nginx and ssl cert**
```
apt install nginx
systemctl start nginx
apt install ssl-cert
```
**Go to `cd /etc/nginx/sites-available/`**

**Make a copy of default so that if you make a mistake you can always go back to the default settings.**

`cp default secure`

**Config secure `nano secure`**
```
Remove both :80 lines
uncomment both :443 lines
```
**Add both these lines below snippets.conf**
```
ssl_certificate /etc/ssl/certs/www.inova.pt.crt;
ssl_certificate_key /etc/ssl/private/www.inova.pt.key;
```
**Change**
```
root /var/www/html;
to ⬇️
root /var/www/htmls;
```
**Add your certificates to location below**
```
/etc/ssl/certs/www.inova.pt.crt
/etc/ssl/private/www.inova.pt.key
```
**Create a soft link of the secure file**
```
cd /etc/nginx/sites-enabled/
ln -s /etc/nginx/sites-available/secure secure
```
**Now check if you have everything working**
```
systemctl restart nginx
systemctl status nginx
```
**And your done with nginx!**

# vsFTPd

**Install vsFTPd**

`apt install vsftpd`

**Add a user**

`adduser client`

**Config your FTP at `nano /etc/vsftpd.conf`**

`Uncomment write_enable=YES`

**Now there are 3 option which you can choose ( Insecure, Explicit and Implicit ).**

**For insecure add this at the bottom**

![image](https://user-images.githubusercontent.com/98783977/154138440-aa38af25-0a4d-4c84-9e49-f42dd1cdddb2.png)

**For explicit add this**

![image](https://user-images.githubusercontent.com/98783977/154138508-f86b0ff9-8553-4a24-b225-e5989dbba646.png)

**Make sure that you certificates are correct and your ssl_enable=YES**
```
rsa_cert_file=/etc/ssl/certs/ftp.inova.pt.crt
rsa_private_key_file=/etc/ssl/private/ftp.inova.pt.key
ssl_enable=YES
```
**For implicit add this**

![image](https://user-images.githubusercontent.com/98783977/154138687-22f95428-1b43-412e-8025-86f8c3677e0a.png)

**Just add those two lines at the bottom**

`systemctl restart vsftpd.service`
