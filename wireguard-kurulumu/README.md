# WireGuard Setup and Configuration Guide

## Server Setup

### WireGuard Installation

```bash
sudo apt install wireguard -y

curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
sudo ./wireguard-install.sh
```

Enter the following information during the installation prompt:

```text
IPv4 or IPv6 Public IP address: [Static_IP_Address]

Server WireGuard IPv4: 10.8.0.1

Server WireGuard port: 51820 (İstediğimizi girebiliriz girdikten sonra diğer yerlerde de aynı portu kullanmalıyız)

Client WireGuard IPv4: 10.8.0.x  (örn 10.8.0.2, 10.8.0.5...)
```

---

## Server Configuration

Open the WireGuard configuration file:

```bash
sudo nano /etc/wireguard/wg0.conf
```

### Add to the [Interface] Section

```ini
PostUp = iptables -I INPUT -p udp --dport 51820 -j ACCEPT; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -i enp3s0 -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp3s0 -j MASQUERADE

PostDown = iptables -D INPUT -p udp --dport 51820 -j ACCEPT; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -i enp3s0 -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp3s0 -j MASQUERADE
```
> [!WARNING]
> Replace the Ethernet interface name `enp3s0` according to your own system.

### Add to the [Peer] Section

```ini
AllowedIPs = 10.8.0.x/32
```
> [!WARNING]
>  10.8.0.x: The specific IP address assigned to that client.

---

## Client (Jetson / Raspberry Pi) Setup

### WireGuard Installation

```bash
sudo apt install wireguard wireguard-tools -y
```

### Client Configuration

Copy the client .conf file generated on the server over to the client and save it to the following path:

```bash
sudo nano /etc/wireguard/client.conf
```

Verify the following parameters:

```ini
Endpoint = [Static_IP_Address]:51820

AllowedIPs = 10.8.0.0/24

PersistentKeepalive = 25
```
If you want to access the remote WireGuard clients (such as the Jetson/Raspberry Pi) from computers without WireGuard installed that are connected via Ethernet to the same router (e.g., GCS PC), you can add the 192.168.10.0/24 network.

## Router Settings

Port Forwarding configuration:

| Setting | Value |
|--------|--------|
| Destination IP | 192.168.1.x (LAN IP address of the WireGuard server) |
| Protocol | UDP |
| Internal Port | 51820 |
| External Port | 51820 |

---

## Starting the Connection

### Server

```bash
sudo wg-quick up wg0
```

### Client

```bash
sudo wg-quick up client
```

### Status Check

```bash
sudo wg show
```
