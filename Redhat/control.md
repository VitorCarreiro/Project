# Control

t2.small

IP:172.31.0.100
   172.31.1.100
   172.31.2.100

Elastic IP: 35.172.35.99

## NAT

**For NAT click on your server go to Actions-Networking-Change source/destination check and press the stop square**

![image](https://user-images.githubusercontent.com/98783977/154124253-1191befb-b8b7-4f15-aca2-73b0adf851ad.png)

**Install iptables and enable it**
```
yum install iptables-services -y
systemctl enable iptables
systemctl start iptables
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
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 587 -j DNAT --to-destination 172.31.1.101
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 110 -j DNAT --to-destination 172.31.1.101
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 143 -j DNAT --to-destination 172.31.1.101
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 993 -j DNAT --to-destination 172.31.1.101
iptables -t nat -A PREROUTING -i eth0 -p tcp -m tcp --dport 995 -j DNAT --to-destination 172.31.1.101
iptables -t nat -A PREROUTING -d 172.29.0.0/16 -i tun-ss -j NETMAP --to 172.31.0.0/16
iptables -t nat -A POSTROUTING -s 172.31.0.0/16 -o tun-ss -j NETMAP --to 172.29.0.0/16
systemctl restart iptables
service iptables save
```
**You need to enable IP forwarding if you want to be capable of forwarding packets to other destinations, go to `nano /etc/sysctl.conf` and add**

`net.ipv4.ip_forward=1`

**Do this to check if its correct then save it with**
```
sysctl -w net.ipv4.ip_forward=1
sysctl -p
```
## DNS

**Install DNS**

`yum install bind`

**Go to `nano /etc/named.conf`, remove and replace those lines**
```
line 13 listen-on port 53 { any; };
line 21 allow-query { any; };
line 35 dnssec-enable no;
line 36 dnssec-validation no;
```
**Go to last line and add above both includes**
```
zone "enta.pt" IN {
        type master;
        file "/var/named/db.enta.pt";

};

zone "31.172.in-addr.arpa" IN {
        type master;
        file "/var/named/db.31.172";

};
```
**Go to the folder `cd /var/named` and copy these**
```
cp named.localhost db.enta.pt
cp named.localhost db.31.172
```
**Change their permissions**
```
chmod named db.enta.pt
chgrp named db.31.172
```
**Config db.enta.pt which is the forward zone to be like this**

![image](https://user-images.githubusercontent.com/98783977/154126788-f8d372fc-2ae9-4f56-b03c-e50fc2057af9.png)

**Config db.31.172 which is the reverse zone to be like this**

![image](https://user-images.githubusercontent.com/98783977/154126961-68e9920a-9dc3-4562-8126-fa1f08d8faa1.png)

**Restart your DNS server**

`systemctl restart named`

## Easy-RSA

**Install the package required to install easy-rsa**

`sudo amazon-linux-extras install epel -y`

**Install easy-rsa**

`yum install easy-rsa`

**Follow this steps that I made since it requires you to use both inova and enta to do this**

https://github.com/VitorCarreiro/CA-Subca

(You shouldn't create the certificates on this server(enta.pt) since only the subca needs to create it, that is because if something happens to the subca, the root can eliminate it or remove the permissions.

## OpenVPN

**Install OpenVPN**

`yum install openvpn`

**Go to `cd /etc/openvpn`**

**Create the Diffie-Hellman pem**

`openssl dhparam -out dh2048.pem 2048`

**Copy an example of the server_ra to /etc/openvpn and change its name**
```
cp /usr/share/doc/openvpn-2.4.11/sample/sample-config-files/server.conf /etc/openvpn/.
mv server.conf server_ra.conf
```
**Copy the certificates that you created on inova to this folder(/etc/openvpn), and copy the ta.key aswell**

**Do `nano server_ra.conf` and change this parameteres**
```
ca ca.crt
cert serverra.enta.pt.crt
key serverra.enta.pt.key
```
**Push routes to the client to allow it to reach other private subnets**
```
push "route 172.31.0.0 255.255.0.0"
push "route 172.31.1.0 255.255.0.0"
push "route 172.31.2.0 255.255.0.0"
push "route 172.31.31.0 255.255.0.0"
```
**Leave and save**

## Teleport

**Install teleport**

**To install and configure teleport you can go to their website since there is everything you need https://goteleport.com/docs/getting-started/linux-server/ 

# Wazuh Agent

**Only do this after your wazuh server is done** ✔️

**For the wazuh agent which is to make you see what improvements you can make to your security overall you have to do those commands**
```
sudo WAZUH_MANAGER='172.31.1.102' WAZUH_AGENT_GROUP='default' yum install https://packages.wazuh.com/4.x/yum/wazuh-agent-4.2.5-1.x86_64.rpm
sudo systemctl daemon-reload
sudo systemctl enable wazuh-agent
sudo systemctl start wazuh-agent
```














