# 4G & Wireguard Kurulum ve Yönetim Rehberi

4G LTE tabanlı bu haberleşme mimarisi, İHA ile yer kontrol istasyonu arasındaki mesafe kısıtını ortadan kaldırarak farklı fiziksel ağlarda bulunan sistemlerin WireGuard VPN üzerinden aynı sanal ağ içerisinde haberleşmesini sağlar.
Bu haberleşme mimarisinde yer istasyonunda bulunan router, statik IP adresine sahip olup internet üzerinden erişilebilen tek noktayı oluşturur. İHA üzerindeki görev bilgisayarı(Jetson/Raspberry Pi), 4G modem/HAT üzerindeki sim kart aracılığıyla internete çıkarak WireGuard client bağlantısını bu statik IP adresine yönlendirir. Dış ağdan router'a ulaşan WireGuard trafiği, port yönlendirme (port forwarding) kuralları sayesinde yerel ağdaki WireGuard sunucusu olarak çalışan bilgisayara iletilir. Böylece CGNAT kaynaklı erişim kısıtlamaları aşılır ve İHA ile yer istasyonu arasında güvenli bir VPN tüneli oluşturulur. Tünelin kurulmasının ardından her iki sistem de aynı sanal ağ içerisinde yer alır ve internet üzerinden haberleşmelerine rağmen birbirleriyle yerel ağdaymış gibi doğrudan, güvenli ve kesintisiz şekilde veri alışverişi yapabilir.
WireGuard sunucusuna bağlı olan yerel ağdaki diğer cihazlar(YKİ PC gibi) ise üzerinde WireGuard kurulu olmasa dahi, routerın yönlendirme ve ağ geçidi yapılandırmaları sayesinde VPN ağına erişebilir ve İHA üzerindeki clientlar ile doğrudan haberleşebilir. Bu sayede yerel ağdaki bilgisayarlar ile İHA arasında telemetri ve görüntü aktarımı işlemleri gerçekleştirilebilir.
---

---<img width="1646" height="1712" alt="RPI_4G_Topology drawio" src="https://github.com/user-attachments/assets/78437a71-6bde-4fca-a026-eba64bb0cb41" />
<img width="1646" height="1712" alt="4G_Topology drawio" src="https://github.com/user-attachments/assets/e4148b1a-7094-4cf7-a5df-82fba8b955b7" />

---
## 🗺️ Hızlı Navigasyon

Aşağıdaki kartları kullanarak gitmek istediğiniz rehbere doğrudan geçiş yapabilirsiniz:

| Bölüm | Bağlantı |
| :--- | :--- |
|Jetson Orin Nano/NX - Sim7600G-H 4G Dongle ve Raspberry Pi 5 - Sixfab 4G Hat Kurulumu | [Görüntüle ➔](./4g-kurulumu/README.md) |
|Wireguard VPN Server ve Client Konfigürasyonları | [Görüntüle ➔](./wireguard-kurulumu/README.md) |
|Genel Kullanım ve Sorun Giderme İçin Önemli Komutlar | [Görüntüle ➔](./komutlar/README.md) |



