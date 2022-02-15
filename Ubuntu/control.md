# Control

t2.small

IP: 172.31.0.100
    172.31.1.100
    172.31.2.100

Elastic IP: 44.240.33.218

## NAT

**For NAT click on your server go to Actions-Networking-Change source/destination check and press the stop square**

![image](https://user-images.githubusercontent.com/98783977/154141517-474276c5-26c0-480e-9eb6-36ba5729cdcd.png)

**Install iptables and netfilter**
```
apt install netfilter-persistent iptables-persistent
netfilter-persistent reload
netfilter-persistent save
```
**Do POSTROUTING MASQUERADE to allow route traffic without disrupting the original one, and add the rest of them for the services we are going to use aswell**
```
iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 80 -j DNAT --to-destination 172.31.2.101
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 443 -j DNAT --to-destination 172.31.2.101
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 444 -j DNAT --to-destination 172.31.1.102:443
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 20 -j DNAT --to-destination 172.31.2.101
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 21 -j DNAT --to-destination 172.31.2.101
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 3389 -j DNAT --to-destination 172.31.1.103
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 3390 -j DNAT --to-destination 172.31.1.104:3389
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 10000:10100 -j DNAT --to-destination 172.31.2.101
iptables -t nat -A PREROUTING -d 172.29.0.0/16 -i tun-ss -j NETMAP --to 172.31.0.0/16
iptables -t nat -A POSTROUTING -s 172.31.0.0/16 -o tun-ss -j NETMAP --to 172.29.0.0/16
systemctl restart iptables
service iptables save
```
**You need to enable IP forwarding if you want to be capable of forwarding packets to other destinations, go to `nano /etc/sysctl.conf` and add**

`net.ipv4.ip_forward=1`

You need to enable IP forwarding if you want to be capable of forwarding packets to other destinations, go to

`nano /etc/sysctl.d/99-sysctl.conf` and uncomment `"net.ipv4.ip_forward=1"`

**Do this to check if its correct then save it with**
```
sysctl -w net.ipv4.ip_forward=1
sysctl -p
```
## DNS

**Install DNS**

`apt install bind9`

**Go to `nano /etc/bind/named.conf.options`, uncomment these 3 lines and add google DNS below forwarders**

![image](https://user-images.githubusercontent.com/98783977/154143206-61979f27-a9ea-42c7-bf83-6bdbf6e887f3.png)

**Go to `cd /etc/bind` and copy these files then move it to `/var/lib/bind`**
```
cp db.local db.inova.pt
cp db.local db.31.172
mv db.inova.pt /var/lib/bind/
mv db.31.172 /var/lib/bind/
```
**Go to `nano /var/lib/bind/db.31.172` and config your reverse zone at the bottom (here are some examples)

![image](https://user-images.githubusercontent.com/98783977/154143581-3b868b84-b314-43b0-82ac-6b256f335317.png)

**Now config your forward zone `nano /var/lib/bind/db.inova.pt`**

![image](https://user-images.githubusercontent.com/98783977/154143726-9d818452-0ab0-4736-b6e9-d7fbf93a0eed.png)

**Go to `nano /etc/bind/named.conf.local`, copy this zones and config your zones if needed**
```
zone "inova.pt" {
        type master;
        file "/var/lib/bind/db.inova.pt";
};

zone "31.172.in-addr.arpa" {
        type master;
        file "/var/lib/bind/db.31.172";
};
```
**Restart you DNS**

`systemctl restart bind9`

**Test your DNS**

`nslookup control.inova.pt`

## Easy-RSA

**Install easy-rsa**

`apt install easy-rsa`

**Follow this steps that I made since it requires you to use both inova and enta to do this**

https://github.com/VitorCarreiro/CA-Subca

**After finishing these steps you can now create your certificates**

**Go to /etc/easy-rsa and create them 1 by 1**
```
./easyrsa build-server-full *.tele.enta.pt nopass
./easyrsa build-server-full tele.enta.pt nopass
./easyrsa build-server-full dovecot.enta.pt nopass
./easyrsa build-server-full dovecot.inova.pt nopass
./easyrsa build-server-full ftp.enta.pt nopass
./easyrsa build-server-full ftp.inova.pt nopass
./easyrsa build-server-full serverra.enta.pt nopass
./easyrsa build-server-full serverra.inova.pt nopass
./easyrsa build-server-full serverss.inova.pt nopass
./easyrsa build-server-full www.inova.pt nopass
./easyrsa build-server-full www.enta.pt nopass
./easyrsa build-client-full clientra.inova.pt nopass
./easyrsa build-client-full clientra.enta.pt nopass
./easyrsa build-client-full clientss.enta.pt nopass
```
## OpenVPN

**Install OpenVPN**

`apt install openvpn`

**Go to `cd /etc/openvpn`**

**Create the Diffie-Hellman pem**

`openssl dhparam -out dh2048.pem 2048`

**Generate the ta.key (copy this ta.key to the control.enta.pt machine**

`openvpn --genkey --secret ta.key`

**Unzip the server.conf of openvpn**
```
cd /usr/share/doc/openvpn/examples/sample-config-files
gzip -d server.conf.gz
```
**Copy the server.conf to /etc/openvpn and rename it two times**

`cp /usr/share/doc/openvpn/examples/sample-config-files/server.conf /etc/openvpn/.`

` cp server.conf server_ra.conf`

` mv server.conf server_ss.conf`

**Config server_ra.conf**

**Place the certificates**
```
ca ca.crt
cert serverra.inova.pt.crt
key serverra.inova.pt.key
```
**And change the push routes**
```
push "route 172.31.0.0 255.255.0.0"
push "route 172.31.1.0 255.255.0.0"
push "route 172.31.2.0 255.255.0.0"
push "route 172.31.31.0 255.255.0.0"
```
**Now config server_ss.conf**

**Place the certificates**
```
ca ca.crt
cert serverss.inova.pt.crt
key serverss.inova.pt.key
```
**Add the remote and route**

![image](https://user-images.githubusercontent.com/98783977/154151667-d0b446ba-ce30-4cb6-b90b-02f69b5b7c0e.png)

**Add push routes**
```
push "route 172.30.0.0 255.255.0.0"
push "route 10.8.0.0 255.255.255.0"
```
**Add tls-server at the end like this**

![image](https://user-images.githubusercontent.com/98783977/154151889-5e05bc4e-469f-4515-851e-2060c989afd8.png)

# Teleport

**To install and configure teleport you can go to their website since there is everything you need https://goteleport.com/docs/getting-started/linux-server/

# Wazuh Agent

**Only do this after your wazuh server is done** ✔️

**For the wazuh agent which is to make you see what improvements you can make to your security overall you have to do those commands**
```
curl -so wazuh-agent-4.2.5.deb https://packages.wazuh.com/4.x/apt/pool/main/w/wazuh-agent/wazuh-agent_4.2.5-1_amd64.deb && sudo WAZUH_MANAGER='172.31.1.102' WAZUH_AGENT_GROUP='default' dpkg -i ./wazuh-agent-4.2.5.deb
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```
