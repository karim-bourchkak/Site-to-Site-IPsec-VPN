# Site-to-Site IPsec VPN in Cisco Packet Tracer

Zwei Standorte (192.168.1.0/24 und 192.168.10.0/24) werden über ein simuliertes WAN mit einem verschlüsselten IPsec-Tunnel verbunden.

## Umgebung

- Cisco Packet Tracer
- 2 Router (Router0 und Router2), 1 Switch, 2 PCs
- IPsec mit ISAKMP (IKE Phase 1) und ESP (Phase 2)

## Netzplan

| Gerät | Schnittstelle | Adresse | Rolle |
|---|---|---|---|
| PC0 | NIC | 192.168.1.10/24 (Gateway 192.168.1.1) | Standort A |
| Router0 | Fa0/0 | 192.168.1.1/24 | LAN Standort A |
| Router0 | Fa0/1 | 200.100.50.1/24 | WAN |
| Router2 | Fa0/0 | 200.100.50.2/24 | WAN |
| Router2 | Fa0/1 | 192.168.10.1/24 | LAN Standort B |
| PC1 | NIC | 192.168.10.10/24 (Gateway 192.168.10.1) | Standort B |

Die Topologie liegt als Bild im Repository: `Network_Topology_Overview.png`.

## Umsetzung

1. Statische Routen: Router0 kennt 192.168.10.0/24 über 200.100.50.2, Router2 kennt 192.168.1.0/24 über 200.100.50.1.
2. ISAKMP-Policy (Phase 1): AES, SHA, Pre-Shared Key, Diffie-Hellman Gruppe 2, Lebensdauer 86400 s.
3. Pre-Shared Key auf beiden Routern, jeweils mit der Adresse des Gegenübers.
4. Transform-Set `MY_TRANSFORM` mit `esp-aes esp-sha-hmac` (Phase 2).
5. Access-Liste 100 definiert den zu verschlüsselnden Verkehr (auf beiden Routern gespiegelt).
6. Crypto Map `MY_MAP` mit Peer, Transform-Set und ACL, angewendet auf die **WAN-Schnittstelle**.

### Konfiguration Router0

```
crypto isakmp policy 10
 encryption aes
 authentication pre-share
 group 2
!
crypto isakmp key <PRE_SHARED_KEY> address 200.100.50.2
!
crypto ipsec transform-set MY_TRANSFORM esp-aes esp-sha-hmac
!
access-list 100 permit ip 192.168.1.0 0.0.0.255 192.168.10.0 0.0.0.255
!
crypto map MY_MAP 10 ipsec-isakmp
 set peer 200.100.50.2
 set transform-set MY_TRANSFORM
 match address 100
!
interface FastEthernet0/1
 crypto map MY_MAP
```

### Konfiguration Router2

```
crypto isakmp policy 10
 encryption aes
 authentication pre-share
 group 2
!
crypto isakmp key <PRE_SHARED_KEY> address 200.100.50.1
!
crypto ipsec transform-set MY_TRANSFORM esp-aes esp-sha-hmac
!
access-list 100 permit ip 192.168.10.0 0.0.0.255 192.168.1.0 0.0.0.255
!
crypto map MY_MAP 10 ipsec-isakmp
 set peer 200.100.50.1
 set transform-set MY_TRANSFORM
 match address 100
!
interface FastEthernet0/0
 crypto map MY_MAP
```

Der Schlüssel ist hier als Platzhalter angegeben und muss auf beiden Routern identisch sein.

## Ergebnis

- **Ping durch den Tunnel:** `ping -n 10 192.168.10.10` von PC0 ergibt 10 von 10 Antworten, im Durchschnitt 12 ms, TTL=126 (die Pakete laufen über zwei Router). Die ersten Pakete gehen beim Tunnelaufbau teilweise verloren, danach funktioniert die Verbindung. Siehe `ping-success.png`.
- **Phase 1:** `show crypto isakmp sa` zeigt den Zustand `QM_IDLE` / `ACTIVE` zwischen 200.100.50.1 und 200.100.50.2. Siehe `isakmp-sa.png`.
- **Phase 2:** `show crypto ipsec sa` zeigt verschlüsselte und entschlüsselte Pakete (`#pkts encaps: 15`, `#pkts decaps: 13`) mit `esp-aes esp-sha-hmac`. Siehe `ipsec-sa.png`.
- **Crypto Map:** `show crypto map` zeigt, dass `MY_MAP` auf Router0 an `FastEthernet0/1` (WAN) hängt. Siehe `Router0_Crypto_Map_Configuration.png` und `Router2_Crypto_Map_Configuration.png`.

## Fehlersuche

Beim ersten Test schlugen alle Pings fehl, obwohl Routing und Policy korrekt waren. Ursache: Auf Router0 war die Crypto Map an `FastEthernet0/0` (LAN) statt an `FastEthernet0/1` (WAN) gebunden. Nach dem Verschieben auf die WAN-Schnittstelle wurde der Tunnel aufgebaut. Vorgehen bei der Eingrenzung: `ping` zum Gateway, dann `show ip route`, `show crypto map`, `show access-lists 100`.

## Grenzen

- Es ist eine Simulation in Packet Tracer, kein produktives Netzwerk.
- AES-128, SHA-1 und Diffie-Hellman Gruppe 2 sind für den Lernzweck gewählt. In der Praxis würde man IKEv2, stärkere DH-Gruppen und SHA-256 verwenden.
- Die Konfiguration nutzt einen Pre-Shared Key, keine Zertifikate.

## Dateien

| Datei | Inhalt |
|---|---|
| `Site-to-Site-IPsec-VPN-Project.pkt` | Packet-Tracer-Projekt |
| `Network_Topology_Overview.png` | Topologie |
| `Router0_Crypto_Map_Configuration.png` | `show crypto map` auf Router0 |
| `Router2_Crypto_Map_Configuration.png` | Konfiguration Router2 |
| `ping-success.png` | Erfolgreicher Ping durch den Tunnel |
| `isakmp-sa.png` | Phase 1: `QM_IDLE` |
| `ipsec-sa.png` | Phase 2: verschlüsselte Pakete |
