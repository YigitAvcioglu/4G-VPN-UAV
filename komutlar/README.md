# Genel Kullanım ve Debugging için Komutlar 

## 4G Komutlar
Sistemdeki tüm ağ arayüzlerini, IP adreslerini ve durumlarını listeler
```bash
ip a
```
Belirtilen ağ arayüzünü (örn: wwan0) yazılımsal olarak kapatır / açar
```bash
ip link set wwan0 down
ip link set wwan0 up
```
4G bağlantısı servis kontrolleri
```bash
systemctl daemon-reload
systemctl status 4g_connect.service
systemctl restart 4g_connect.service
systemctl stop 4g_connect.service
systemctl enable 4g_connect.service
systemctl disable 4g_connect.service
```
Jetson için seri port üzerinden modemin AT komut arayüzüne bağlanır
```bash
sudo minicom -D /dev/ttyUSB2
```
Jetson için minicom içinde çalıştırılan AT komutları
```bash
AT                    # Modemle bağlantıyı doğrular (OK döner)
AT+CPIN?              # SIM kartın pin durumunu ve takılı olup olmadığını sorgular
AT+CSQ                # Sinyal gücünü ölçer (0-31 arası döner, 15+ idealdir)
AT+CREG?              # Şebeke kayıt durumunu doğrular (1=Ev Şebekesi, 5=Roaming)
AT+COPS?              # Bağlı olunan operatör ismini (Turkcell, Vodafone vb.) gösterir
AT+CNMP=2             # Otomatik mod (Fallback): 4G koparsa otomatik 3G/2G'ye düşer
AT+CNMP=38            # Sadece 4G modu: Modemi sadece LTE baz istasyonlarına kilitler
AT+CUSBPIDSWITCH=9001 # Modülü QMI (Gelişmiş Hücresel Tünel) moduna geçirir
AT$QCRMCALL=1,1       # QMI modunda hücresel tüneli ve veri akışını elle başlatır
```

## Wireguard Komutları

Wireguard tünelini açar/kapatır
```bash
sudo wg-quick up wg0
sudo wg-quick down wg0

sudo wg-quick up client
sudo wg-quick down client
```
Wireguard durumunu gösterir
```bash
sudo wg show
```
Canlı ICMP (Ping) Trafiği İzleme
```bash
sudo tcpdump -n -i wg0 icmp

sudo tcpdump -n -i client icmp
```
O anki aktif tüm rota tablosunu listeler
```bash
ip route show
```
İnternete hangi karttan ve hangi gateway üzerinden çıkıldığını gösterir
```bash
ip route | grep default
```
