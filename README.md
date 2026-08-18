


# 4G & WireGuard Setup and Management Guide

This 4G LTE-based communication architecture eliminates the range limitation between the UAV (Unmanned Aerial Vehicle) and the Ground Control Station (GCS), enabling systems located on different physical networks to communicate within the same virtual network via WireGuard VPN.

In this communication architecture, the router located at the ground station has a static IP address, serving as the single access point reachable over the internet. The mission computer on the UAV (Jetson / Raspberry Pi) accesses the internet via the SIM card on the 4G modem/HAT and directs the WireGuard client connection to this static IP address. The incoming WireGuard traffic reaching the router from the external network is forwarded to the computer acting as the WireGuard server on the local network via port forwarding rules. In this way, CGNAT-related access restrictions are bypassed, and a VPN tunnel is established between the UAV and the ground station. Once the tunnel is established, both systems reside within the same virtual network; despite communicating over the public internet, they can exchange data directly, securely (encrypted), and seamlessly as if they were on the same local network.

Furthermore, other devices on the local network connected to the WireGuard server (such as the GCS PC) can access the VPN network and communicate directly with the clients on the UAV—even if WireGuard is not installed on them—thanks to the router's routing and gateway configurations. This enables telemetry and video transmission between computers on the local network and the UAV.

---
https://github.com/user-attachments/assets/91979a94-be5c-4602-8060-16c8fb7bc293

---
## Quick NAvigation

You can jump directly to the relevant guide using the links below:

| Section | Link |
| :--- | :--- |
|Jetson Orin Nano/NX - Sim7600G-H 4G Dongle and Raspberry Pi 5 - Sixfab 4G Hat Setup | [View ➔](./4g-kurulumu/README.md) |
|Wireguard VPN Server ve Client Configurations | [View ➔](./wireguard-kurulumu/README.md) |
|Essential Commands for General Usage and Troubleshooting | [View ➔](./komutlar/README.md) |



