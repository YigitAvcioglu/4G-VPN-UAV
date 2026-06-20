# 4G Cihazları Kurulumu
---
## Jetson Orin Nano/NX - Sim7600G-H 4G Dongle Kurulumu
---
### Gereklilikleri indirin
```bash
sudo apt install -y build-essential linux-headers-$(uname -r) minicom udhcpc wget unzip iproute2 coreutils gawk grep
```
---
### Sürücüyü kurun
```bash

cd Documents
wget https://files.waveshare.com/upload/4/46/Simcom_wwan.zip
unzip Simcom_wwan.zip
cd Simcom_wwan
sudo make
```
---
### Otomatik bağlantı scripti oluşturun
```bash
sudo nano 4g_connect.sh
```
Bunu dosyanın içine yapıştırın
```bash
#!/bin/bash

# 1. ModemManager'ı kapat (Portları kilitlememesi için)
systemctl stop ModemManager 2>/dev/null

# 2. Sürücü Temizliği ve Kendi Derlediğimiz Sürücüyü Yükleme
rmmod simcom_wwan 2>/dev/null
rmmod qmi_wwan 2>/dev/null
sleep 2

modprobe usbnet cdc_ether 2>/dev/null
insmod ~/Downloads/simcom_wwan/simcom_wwan.ko 2>/dev/null
sleep 5

# 3. Port Tespiti
TARGET_PORT="/dev/ttyUSB2"
if [ ! -c "$TARGET_PORT" ]; then
    TARGET_PORT="/dev/ttyUSB3"
fi

# 4. Modemin komut almaya hazır olmasını bekle
stty -F $TARGET_PORT 115200 raw -echo -echoe -echok -echoctl -echoke 2>/dev/null
sleep 1

# 5. APN Tanımlama
exec 3<>$TARGET_PORT
printf "AT+CGDCONT=1,\"IP\",\"internet\"\r\n" >&3
sleep 2

# 6. QMI Tüneli 
printf "AT\$QCRMCALL=1,1\r\n" >&3
sleep 5
exec 3>&- 

# 7. Dinamik Arayüz Tespiti
INTERFACE=$(ip -o link show | awk -F': ' '{print $2}' | grep -E '^wwan|^usb' | tail -n 1)
if [ -z "$INTERFACE" ]; then
    INTERFACE="wwan0"
fi

# 8. Kartı Up Yap ve Çakışan Eski Rotaları Temizle
ip link set $INTERFACE down 2>/dev/null
sleep 1
ip link set $INTERFACE up
sleep 2
# Mevcut internet ağları DHCP'yi bloklamasın diye varsayılan rotayı sil
ip route del default 2>/dev/null

# 9. IP Alma
echo "IP talep ediliyor..."
udhcpc -i $INTERFACE -b -R 5

echo "4G Dongle ile internete bağlanıldı"
```
---
### Servis dosyası oluşturun
```bash
sudo nano 4g_connect.service
```
Bunu dosyanın içine yapıştırın
```bash
[Unit]
Description=Hücresel Ağ Otomatik Bağlantı Servisi
After=network.target
Before=rc-local.service

[Service]
Type=oneshot
ExecStart=/usr/local/bin/4g_connect.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```
Servis dosyasını başlangıçta çalışacak şekilde ayarla ve başlat
```bash
sudo systemctl daemon-reload
sudo systemctl enable 4g_connect.service
sudo systemctl start 4g_connect.service
```
---
## Raspberry Pi - Sixfab 4G HAT - Quectel25-EUX Kurulumu
```
### Donanımı Hazırlayın
1. 4g Hat'i Raspberry Pi'a takın
2. Sim Kartı Takın (Sim Kartın şifresi kapalı olmalı)
3. Micro USB kablosunu takın
4. Gücü açın

Cihaz görünüyor mu ve portlar oluştu mu?
```bash
lsusb
dmesg | grep ttyUSB
```

### Yazılımsal Kurulum
```bash
sudo apt update
sudo apt install unzip build-essential -y
cd ~
wget https://sixfab.com/wp-content/uploads/2023/09/Quectel_QConnectManager_Linux_V1.6.5.zip
unzip -q Quectel_QConnectManager_Linux_V1.6.5.zip
mv Quectel_QConnectManager_Linux_V1.6.5 quectel-cm
cd quectel-cm
make
```
Bağlantıyı başlatın
```bash
sudo ./quectel-cm -s internet &
```
"internet" yerine operatörün APN adı yazılır. Turkcell için APN adı "internet"

IP adresi kontrolü 
```bash
ip a show wwan0
```
Eğer IP alındıysa bağlantı başarılı

## Servis dosyasını oluşturalım
```bash
sudo nano /etc/systemd/system/4g_connect.service
```
```bash
[Unit]
Description=Quectel 4G LTE Connection Manager
After=network.target

[Service]
Type=simple
ExecStart=/home/kullanıcı_adı/quectel-cm/quectel-cm -s internet
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Başlangıçta çalışacak şekilde ayarlayalım ve başlatalım
```bash
sudo systemctl daemon-reload
sudo systemctl enable 4g_connect.service
sudo systemctl start 4g_connect.service
```
