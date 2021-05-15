---
title: Setting Up WireGuard Server and Client on Ubuntu 18.04
created: '2021-05-14T20:19:54.186Z'
modified: '2021-05-15T09:57:57.310Z'
---

# Setting Up WireGuard Server and Client on Ubuntu 18.04
## Server Setup
### Step 1 : Install WireGuard on the Server
To install WireGuard on your server, just copy and paste the following command inside your terminal
```sh
sudo apt install wireguard
```

### Step 2 : Generate Certificates
To proceed, you'll need public and private key certificates of the server. To obtain those, execute these commands
```sh
umask 077
wg genkey | tee server_private_key | wg pubkey > server_public_key
```

### Step 3 : Generate Server Config
Create the server configuration file ***(/etc/wireguard/wg0.conf)*** using the template provided here.
```sh
[Interface]
Address = 10.100.100.1/24
SaveConfig = true
PrivateKey = <SERVER_PRIVATE_KEY>
ListenPort = 51820
PostUp = iptables -A FORWARD -i %i -j ACCEPT; iptables -A FORWARD -o %i -j ACCEPT; iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i %i -j ACCEPT; iptables -D FORWARD -o %i -j ACCEPT; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

[Peer]
PublicKey = <CLIENT_PUBLIC KEY> //This one is yet to be generated
AllowedIPs = 10.100.100.2/32
```
**NOTE:** If your server is behind NAT (eg. Oracle Cloud VMs), the above settings will not work for you, you can use below config instead. Please note that you'll have to replace **eth0** with the name if your network card, you can get it by running **ifconfig** inside the machine.
```sh
[Interface]
Address = 10.100.100.1/24
SaveConfig = true
PrivateKey = <SERVER_PRIVATE_KEY>
ListenPort = 51820
PreUp = iptables --table nat --append POSTROUTING --jump MASQUERADE --out-interface eth0
PreDown = iptables --table nat --delete POSTROUTING --jump MASQUERADE --out-interface eth0

[Peer]
PublicKey = <CLIENT_PUBLIC KEY> //This one is yet to be generated
AllowedIPs = 10.100.100.2/32
```

### Step 4: Enable IPv4 Forwarding
Enable IPv4 forwarding so that we can access the rest of the LAN and not just the server itself.
Open **/etc/sysctl.conf** and comment out the following lines.
```sh
net.ipv4.ip_forward=1
net.ipv6.conf.all.forwarding=1
```
Now, you can either restart the server, or use the following commands for the IP forwarding to take effect without restarting the server
```sh
sysctl -p
echo 1 > /proc/sys/net/ipv4/ip_forward
```
### Step 5: Start WireGuard
> **NOTE** : Before going for this step, please finish get the client's public key from the second section of this tutorial (Client Setup) and place it inside the **Peer** section of the server's config file.

Run the following commands to start the WireGuard Server Process and add it to auto-start.
```sh
chown -v root:root /etc/wireguard/wg0.conf
chmod -v 600 /etc/wireguard/wg0.conf
wg-quick up wg0
systemctl enable wg-quick@wg0.service 
```

## Client Setup
### Step 1 : Install WireGuard on the Client
WireGuard installation is exactly the same for client. Just copy and paste the following commnad in the terminal
```sh
sudo apt install wireguard
```

### Step 2 : Generate Certificates
As the server, client also needs to generate the certificates to negotiate the connection.
```sh
wg genkey | tee client_private_key | wg pubkey > client_public_key
```
> **NOTE** : Now, copy the client public key and paste it into the server and execute the commands from section 4 from the server section.

### Step 3 : Create Client Config
Create the client configuration file (**/etc/wireguard/wg0-client.conf**) using the template provided here.
```sh
[Interface]
Address = 10.100.100.2/32
PrivateKey = <CLIENT_PRIVATE_KEY>

[Peer]
PublicKey = <SERVER_PUBLIC_KEY>
Endpoint = <SERVER_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 21
```

Step 4 : Start WireGuard
Finally, you can start the WireGuard session with the tunnel config we just created. To do so, execute following command:
```sh
sudo wg-quick up wg0-client
```
