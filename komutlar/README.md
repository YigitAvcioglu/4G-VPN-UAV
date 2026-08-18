# Essential Commands for General Usage and Debugging

## 4G Commands
Lists all network interfaces, IP addresses, and their current states in the system
```bash
ip a
```
Disables or enables the specified network interface (e.g., wwan0)
```bash
ip link set wwan0 down
ip link set wwan0 up
```
4G connection service management
```bash
systemctl daemon-reload
systemctl status 4g_connect.service
systemctl restart 4g_connect.service
systemctl stop 4g_connect.service
systemctl enable 4g_connect.service
systemctl disable 4g_connect.service
```
Connects to the modem's AT command interface over the serial port
```bash
sudo minicom -D /dev/ttyUSB2
```
AT commands executed inside minicom
```bash
AT                    # Verifies communication with the modem (returns OK)
AT+CPIN?              # Queries SIM card PIN status and verifies if it is inserted
AT+CSQ                # Measures signal strength (returns 0-31; 15+ is ideal)
AT+CREG?              # Verifies network registration status (1=Home Network, 5=Roaming)
AT+COPS?              # Displays the connected carrier/operator name
AT+CNMP=2             # Automatic mode (Fallback): Falls back to 3G/2G if 4G disconnects
AT+CNMP=38            # 4G LTE only mode: Locks the modem strictly to LTE base stations
AT+CUSBPIDSWITCH=9001 # Switches the module to QMI (Advanced Cellular Tunnel) mode
AT$QCRMCALL=1,1       # Manually starts the cellular tunnel and data flow in QMI mode
```

## Wireguard Commands

Brings up / brings down the WireGuard tunnel
```bash
sudo wg-quick up wg0
sudo wg-quick down wg0

sudo wg-quick up client
sudo wg-quick down client
```
Displays the current WireGuard status and active handshakes
```bash
sudo wg show
```
Monitors live ICMP (Ping) traffic over the tunnel
```bash
sudo tcpdump -n -i wg0 icmp

sudo tcpdump -n -i client icmp
```
Lists the current active routing table
```bash
ip route show
```
Shows the default interface and gateway used for internet egress
```bash
ip route | grep default
```
