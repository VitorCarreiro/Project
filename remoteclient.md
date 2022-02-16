# Remote Client

IP:172.31.34.68

Elastic IP: 44.237.247.80

**Update and upgrade**

`apt update && apt upgrade -y`

`sudo su -`

`sudo apt install -y xfce4 xfce4-goodies`

`sudo apt install -y xrdp chromium-browser filezilla`

`sudo adduser xrdp ssl-cert`

`echo xfce4-session > ~/.xsession ` **(Leave root)**

`sudo reboot`

`sudo passwd ubuntu`

`adduser client`

# Install OpenVPN

`Go to cd /etc/openvpn`

**Get the ta.key from Inova control**

`cd /usr/share/doc/openvpn/examples/sample-config-files/`

`cp client.conf /etc/openvpn/.`

`cp client.conf client2.conf`

**Config client.conf for inova.pt**

**Change to public IP of inova.pt**

![image](https://user-images.githubusercontent.com/98783977/154172377-7d3afb14-11a3-4070-b5a7-faa583f69815.png)

**Change certificates**
```
ca ca.crt
cert clientra.inova.pt.crt
key clientra.inova.pt.key
```
**Config client2.conf for enta.pt**

**Change to Public IP of enta.pt**

![image](https://user-images.githubusercontent.com/98783977/154172544-c13053f6-c9f6-4260-852c-4eef7b720db2.png)

**Change certificates**
```
ca ca.crt
cert clientra.enta.pt.crt
key clientra.enta.pt.key
```
**Enable client configs**

`systemctl enable --now openvpn@client`
`systemctl enable --now openvpn@client2`

