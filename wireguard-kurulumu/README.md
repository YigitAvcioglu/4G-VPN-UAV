# WireGuard Kurulum ve Yapılandırma Rehberi

## Sunucu Kurulumu

### WireGuard Kurulumu

```bash
sudo apt install wireguard -y

curl -O https://raw.githubusercontent.com/angristan/wireguard-install/master/wireguard-install.sh
chmod +x wireguard-install.sh
sudo ./wireguard-install.sh
```

Kurulum sırasında aşağıdaki bilgileri girin:

```text
IPv4 or IPv6 Public IP address: [Static_IP_Address]

Server WireGuard IPv4: 10.8.0.1

Server WireGuard port: 51820 (İstediğimizi girebiliriz girdikten sonra diğer yerlerde de aynı portu kullanmalıyız)

Client WireGuard IPv4: 10.8.0.x  (örn 10.8.0.2, 10.8.0.5...)
```

---

## Sunucu Yapılandırması

WireGuard yapılandırma dosyasını açın:

```bash
sudo nano /etc/wireguard/wg0.conf
```

### `[Interface]` Bölümüne Ekleyin

```ini
PostUp = iptables -I INPUT -p udp --dport 51820 -j ACCEPT; iptables -A FORWARD -i wg0 -j ACCEPT; iptables -A FORWARD -i enp3s0 -o wg0 -j ACCEPT; iptables -t nat -A POSTROUTING -o enp3s0 -j MASQUERADE

PostDown = iptables -D INPUT -p udp --dport 51820 -j ACCEPT; iptables -D FORWARD -i wg0 -j ACCEPT; iptables -D FORWARD -i enp3s0 -o wg0 -j ACCEPT; iptables -t nat -D POSTROUTING -o enp3s0 -j MASQUERADE
```
> [!WARNING]
> `enp3s0` ethernet arayüzü adınızı kendi sisteminize göre değiştirin.

### `[Peer]` Bölümüne Ekleyin

```ini
AllowedIPs = 10.8.0.x/32
```
> [!WARNING]
>  10.8.0.x: o istemciye atanmış olan IP adresi

---

## İstemci (Jetson / Raspberry Pi) Kurulumu

### WireGuard Kurulumu

```bash
sudo apt install wireguard wireguard-tools -y
```

### İstemci Konfigürasyonu

Sunucuda oluşturulan istemci `.conf` dosyasını istemciye kopyalayın ve aşağıdaki dosyaya kaydedin:

```bash
sudo nano /etc/wireguard/client.conf
```

Aşağıdaki ayarları kontrol edin:

```ini
Endpoint = [Static_IP_Address]:51820

AllowedIPs = 10.8.0.0/24

PersistentKeepalive = 25
```
Eğer WireGuard kurulu olmayan ancak aynı router'a Ethernet ile bağlı olan bilgisayarlarla(YKİ bilgisayarı gibi) uzaktaki wireguard istemcilerine(Jetson/Raspberry Pi) erişmek istiyorsanız `192.168.1.0/24` ağını ekleyebilirsiniz.
> [!WARNING]
>  Ancak bu eklemeden sonra istemci cihazlara(Jetson/Raspberry Pi) ethernet ile SSH atılamaz.

---

## WireGuard Kurulu Olmayan PC İçin

Windows üzerinde statik rota eklemek için(Bu komutu girmeden önce cmd'yi yönetici olarak çalıştırın)
```cmd
route -p ADD 10.8.0.0 MASK 255.255.255.0 192.168.1.x
```
Windows üzerinde statik rota eklemek için:
```cmd
sudo ip route add 10.8.0.0/24 via 192.168.1.x
```
> [!WARNING]
> 192.168.1.x(Wireguard Sunucu PC IP)
> > [!WARNING]
> Firewall'u devre dışı bırakın.

---

## Router Ayarları

Port yönlendirme (Port Forwarding) yapılandırması:

| Ayar | Değer |
|--------|--------|
| Hedef IP | 192.168.1.x (WireGuard sunucusunun LAN IP adresi) |
| Protokol | UDP |
| Internal Port | 51820 |
| External Port | 51820 |

---

## Bağlantıyı Başlatma

### Sunucu

```bash
sudo wg-quick up wg0
```

### İstemci

```bash
sudo wg-quick up client
```

### Durum Kontrolü

```bash
sudo wg show
```
