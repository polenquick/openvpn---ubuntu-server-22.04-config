<h1>How to install openvpn (current version 2.55 - 10/2022)<br></h1>
<code>apt install openvpn easy-rsa</code><br>
<code>make-cadir /etc/openvpn/easy-rsa</code>

<h1>Generate server keys and certificates<br></h1>
<code>cd /etc/openvpn/easy-rsa</code><br>
<code>cd./easyrsa init-pki</code><br>
<code>cd./easyrsa build-ca</code><br>
<code>cd./easyrsa gen-req yourservername nopass</code><br>
<code>cd./easyrsa gen-dh</code><br>
<code>cd./easyrsa sign-req server myservername</code><br>
<code>cp pki/dh.pem pki/ca.crt pki/issued/yourservername.crt pki/private/yourservername.key /etc/openvpn/ </code>

<h2>advanced server conf </h2><br>
<code> cp /usr/share/doc/openvpn/examples/sample-config-files/yourservername.conf /etc/openvpn/yourservername.conf</code><br><br>
Edit /etc/openvpn/yourservername.conf to make sure the following lines are pointing to the certificates and keys you created in the section above.<br>

```
server.conf
local YOUR_SERVER_IP
port 1194
proto udp
dev tun
ca ca.crt
cert yourservername.crt
key yourservername.key # This file should be kept secret
dh dh.pem # or dh2048.pem/dh4096.pem/...
topology subnet
server 10.0.0.0 255.255.255.0
ifconfig-pool-persist /var/log/openvpn/ipp.txt
client-to-client
keepalive 10 120
tls-auth ta.key 0 # This file is secret
cipher AES-256-CBC
data-ciphers AES-256-GCM:AES-128-GCM:AES-256-CBC
persist-key
persist-tun
status /var/log/openvpn/openvpn-status.log
verb 3
explicit-exit-notify 1
```
Now u can generate ta.key in etc/openvpn <br>
<code>openvpn --genkey secret ta.key</code><br><br>
Next step is uncomment the following line to enable IP forwarding. Just remove "#" from that line <br> ```#net.ipv4.ip_forward=1 > net.ipv4.ip_forward=1``` <br> using
<code>nano /etc/sysctl.conf</code><br>
and after this <br>
<code>sudo sysctl -p /etc/sysctl.conf</code>

Now u can start your openvpn server <br>
<code>systemctl start openvpn@yourservername</code><br>
You will find logging and error messages in the journal <br>
<code>journalctl -xeu openvpn@yourservername</code>



<h1>Generate CLIENT keys and certificates<br></h1>
<code>./easyrsa gen-req yourclientname nopass</code><br>
<code>./easyrsa sign-req client yourclientname</code><br>

U must have to copy generated .key, .crt and .config to /etc/openvpn 
```
cp /etc/openvpn/easyrsa/issued/yourclientname.crt /etc/openvpn
cp /etc/openvpn/easyrsa/private/yourclientname.key /etc/openvpn
cp /usr/share/doc/openvpn/examples/sample-config-files/client.conf /etc/openvpn/
```

Edit /etc/openvpn/client.conf to make sure the following lines are pointing to those files.<br>
<code>nano client.conf</code><br>

```
client
dev tun
proto udp
remote YOUR_PUBLIC_IP 1194
;remote-random
resolv-retry infinite
;nobind
;user nobody
;group nobody
persist-key
persist-tun
ca ca.crt
cert yourclientname.crt
key yourclientname.key
remote-cert-tls server
cipher AES-256-CBC
data-ciphers AES-256-GCM:AES-128-GCM:AES-256-CBC
verb 3
```
Now u can start your openvpn client <br>
<code>systemctl start openvpn@client</code><br>
You will find logging and error messages in the journal <br>
<code>journalctl -xeu openvpn@client</code>

Now copy yourclientname.crt & yourclientname.key from server to your desktop <br>
U can use programs like WINSCP on windows and next import to OPENVPN
u'll be need root permision or type
```
chmod +rwx yourclientname.crt
chmod +rwx yourclientname.key
```
![openvpn_1](https://user-images.githubusercontent.com/15954857/195981440-521859c6-3cff-4694-a163-dbe48b8fd8d0.jpg)

