# Site-to-Site IPsec VPN Implementation in Cisco Packet Tracer

This repository contains the configuration files and documentation for a secure **Site-to-Site IPsec VPN** project developed as part of vocational training (*Ausbildung*).

## 🛠️ Project Overview
The main objective of this project is to establish a secure, encrypted communication tunnel between a corporate network and a remote site over an insecure WAN connection using Cisco routers.

* **Topology Components**: 
  * 2x Cisco 2811 Routers (`Router0`, `Router2`)
  * 1x Cisco 2960 Switch (`Switch0`)
  * 2x End Devices (`PC0`, `PC1`)
* **Subnets**:
  * Remote Network: `192.168.1.0/24`
  * Corporate Network: `192.168.10.0/24`
  * WAN Network: `200.100.50.0/24`

## 🔒 Security Policies & Configuration
* **Phase 1 (ISAKMP)**:
  * Policy: `10`
  * Encryption: `AES`
  * Authentication: `Pre-share` (`cisco123`)
  * Diffie-Hellman Group: `2`
* **Phase 2 (IPsec)**:
  * Transform Set: `MY_TRANSFORM` (`esp-aes`, `esp-sha-hmac`)
  * Access Control List: `ACL 100` for traffic matching
  * Crypto Map: `MY_MAP` bound to the WAN interface (`FastEthernet0/0`)

## 📁 Repository Structure
* `Network_Topology_Overview.png`: Visual layout of the network topology in Cisco Packet Tracer.
* `Router0_Crypto_Map_Configuration.png`: CLI output confirming the crypto map configuration on Router0.
* `Router2_Crypto_Map_Configuration.png`: CLI output showing ISAKMP activation and crypto map binding on Router2.
* `Site-to-Site-IPsec-VPN-Project.pkt`: The complete Cisco Packet Tracer simulation file.

## 🚀 How to Use
1. Download and open the `.pkt` file using **Cisco Packet Tracer**.
2. Inspect the router CLI configurations using `show running-config`.
3. Verify the security associations using `show crypto ipsec sa`.
