# Remote-VPN
## pfSense OpenVPN — Remote Access (SSL/TLS + User Auth)

A pfSense-based Remote Access VPN that lets individual client devices connect securely to the internal LAN. 
Authentication is layered: 
1. clients must present a valid certificate (SSL/TLS) and a local username/password, on top of a shared TLS key for control-channel HMAC authentication.

<img width="764" height="609" alt="image" src="https://github.com/user-attachments/assets/3d1ee3ca-5181-4194-b4ec-4da9af099b6c" />

Clients connect over UDP/1194 to the pfSense WAN interface, receive an address from the 10.8.0.0/24 tunnel pool, and are routed into the 192.168.1.0/24 LAN.

<img width="669" height="663" alt="image" src="https://github.com/user-attachments/assets/b9d0d16e-e190-4aad-ba7a-9f62193a3b51" />

## Setup steps
1. Certificate Authority imported an existing CA, myvpn-ca (System > Cert. Manager > Authorities).
2. Server certificate imported opnvpn-server-cert, signed by myvpn-ca (Subject: CN=openvpn, C=IN).
3. OpenVPN server — VPN > OpenVPN > Servers > Add:
Server mode Remote Access (SSL/TLS + User Auth), backend Local Database.
tun device mode, UDP/IPv4, WAN interface, local port 1194.
Enabled a TLS key for HMAC control-channel authentication.
Selected the CA and server certificate above.
Data encryption AES-128-CBC, auth digest SHA256.
Tunnel network 10.8.0.0/24; local network 192.168.1.0/24 advertised to connecting clients.
4. Local user — System > User Manager > Users: created vpnuser1 for VPN login (in addition to certificate auth).
5. Firewall — allowed inbound UDP/1194 on WAN, plus traffic on the resulting OpenVPN interface, so tunneled traffic can reach the LAN.
6. Client — connected with the OpenVPN Connect app.

## Verification

The OpenVPN Connect client shows a "Securely Connected!" session to the pfSense gateway (192.168.31.141), confirming the tunnel establishes and authenticates correctly.

## Notes / possible next steps
1. Use the Client Export tab in pfSense to auto-generate .ovpn / installer bundles instead of configuring clients by hand.
2. Add Client Specific Overrides to assign static tunnel IPs or per-user routes.
3. Consider moving from AES-128-CBC to an AEAD cipher (e.g. AES-256-GCM) for stronger authenticated encryption.
