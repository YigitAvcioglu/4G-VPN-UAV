# 4G Devices Setup
---
## Jetson Orin Nano/NX - Sim7600G-H 4G Dongle Setup
---
### Install the Reuqirements
```bash
sudo apt install -y build-essential linux-headers-$(uname -r) minicom udhcpc wget unzip iproute2 coreutils gawk grep
```
---
### Install the Driver
```bash

cd Documents
wget https://files.waveshare.com/upload/4/46/Simcom_wwan.zip
unzip Simcom_wwan.zip
cd Simcom_wwan
sudo make
```
---
### Create the Auto-Connect Script
```bash
cd
sudo nano 4g_connect.sh
cp 4g_connect.sh /usr/local/bin/4g_connect.sh

```
Paste the following into the file:
```bash
#!/bin/bash

# 1. ModemManager'ı kapat (Portları kilitlememesi için)
systemctl stop ModemManager 2>/dev/null

# 2. Port Tespiti
TARGET_PORT="/dev/ttyUSB2"
if [ ! -c "$TARGET_PORT" ]; then
    TARGET_PORT="/dev/ttyUSB3"
fi

# 3. Modemin komut almaya hazır olmasını bekle
stty -F $TARGET_PORT 115200 raw -echo -echoe -echok -echoctl -echoke 2>/dev/null
sleep 1

# 4. APN Tanımlama
exec 3<>$TARGET_PORT
printf "AT+CGDCONT=1,\"IP\",\"internet\"\r\n" >&3
sleep 2

# 5. QMI Tüneli 
printf "AT\$QCRMCALL=1,1\r\n" >&3
sleep 5
exec 3>&- 

# 6. Dinamik Arayüz Tespiti
INTERFACE=$(ip -o link show | awk -F': ' '{print $2}' | grep -E '^wwan|^usb' | tail -n 1)
if [ -z "$INTERFACE" ]; then
    INTERFACE="wwan0"
fi

# 7. Kartı Up Yap ve Çakışan Eski Rotaları Temizle
ip link set $INTERFACE down 2>/dev/null
sleep 1
ip link set $INTERFACE up
sleep 2
# Mevcut internet ağları DHCP'yi bloklamasın diye varsayılan rotayı sil
ip route del default 2>/dev/null

# 8. IP Alma
echo "IP talep ediliyor..."
udhcpc -i $INTERFACE -b -R 5

echo "4G Dongle ile internete bağlanıldı"
```
### Make the script executable
```bash
sudo chmod +x /usr/local/bin/4g_connect.sh
```

---
### Create the Service File
```bash
sudo nano 4g_connect.service
sudo cp 4g_connect.service /etc/systemd/system/g_connect.service
```
Paste the following into the file:
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
Enable the service to start at boot and start it:
```bash
sudo systemctl daemon-reload
sudo systemctl enable 4g_connect.service
sudo systemctl start 4g_connect.service
```
---
## Raspberry Pi - Sixfab 4G HAT - Quectel25-EUX Setup

### Hardware Preparation
```bash
1. Mount the 4G HAT onto the Raspberry Pi.
2. Insert the SIM Card (SIM Card PIN lock must be disabled).
3. Connect the Micro USB cable.
4. Power on the device.
```

Check if the device is detected and ports are created
```bash
lsusb
dmesg | grep ttyUSB
```

### Software Preparation
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
Start the connection
```bash
sudo ./quectel-cm -s internet &
```
Replace "internet" with your carrier's APN name
Check the IP address
```bash
ip a show wwan0
```
If an IP address is assigned, the connection is successful
## Create the Service File
```bash
sudo nano 4g_connect.service
sudo cp 4g_connect.service /etc/systemd/system/g_connect.service
```
```bash
[Unit]
Description=Quectel 4G LTE Connection Manager
After=network.target

[Service]
Type=simple
ExecStart=/home/user_name/quectel-cm/quectel-cm -s internet
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

Enable the service to start at boot and start it
```bash
sudo systemctl daemon-reload
sudo systemctl enable 4g_connect.service
sudo systemctl start 4g_connect.service
```
