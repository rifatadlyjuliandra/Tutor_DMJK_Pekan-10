# NusantaraNet — Konfigurasi Lengkap X = 1

**SI2514011 | Cisco Packet Tracer 8.2+** · Pasangan berkas: `nusantaranet-X1-checkpoint-viva.md`

## Daftar isi

1. [Cara pakai & catatan kejujuran](#1-cara-pakai--catatan-kejujuran)
2. [Parameter X = 1](#2-parameter-x--1)
3. [Topologi & kabel](#3-topologi--kabel)
4. [Lab pekan 1 dan 2](#4-lab-pekan-1-dan-2-sementara)
5. [Router](#5-router): [R-HQ-1](#r-hq-1) · [R-BR-1](#r-br-1) · [R-GD-1](#r-gd-1) · [ISP-1](#isp-1) · [ISP-2](#isp-2)
6. [Switch](#6-switch): [SW-HQ-1-01](#sw-hq-1-01) · [SW-HQ-1-02](#sw-hq-1-02) · [SW-BR-1-01](#sw-br-1-01) · [SW-GD-1-01](#sw-gd-1-01)
7. [Server, PC, WLAN, IoT](#7-server-pc-wlan-iot-gui)

---

## 1. Cara pakai & catatan kejujuran

Setiap perangkat dibagi menjadi tahap **[P2] [P3] [P4] [P5] [P6] [P10]**. Terapkan berurutan sesuai pekan supaya checkpoint tiap pekan bisa didemonstrasikan (mis. ping ke Server hanya berhasil **sebelum** ACL pekan 6). Setelah tiap tahap: `write memory`.

> **Belum saya uji di Packet Tracer Anda:**
> 1. Router 2911 hanya punya 3 port Gi, sedangkan R-HQ butuh 5 tautan (LAN, Cabang, Gudang, ISP-1, ISP-2). Solusi: tambah modul **HWIC-2T** (power off router → seret HWIC-2T ke slot → power on) → `Se0/0/0` (ISP-1) dan `Se0/0/1` (ISP-2). Router ISP juga diberi HWIC-2T.
> 2. NAT berbasis **route-map** (pekan 10) bisa saja belum didukung PT 8.2. Cek dengan Prompt B. Fallback ada di [R-HQ-1 [P10]](#p10-isp-2-failover-nat-route-map-port-forwarding-acl-luar).
> 3. **AP-PT** hanya memetakan satu SSID ke satu VLAN, jadi di sini 1 AP per SSID per lokasi, uplink berupa port akses (bukan trunk). Jika AP Anda mendukung multi-SSID+VLAN, ganti uplink menjadi trunk.
> 4. `ip sla` / `track` (deteksi upstream) kemungkinan tidak ada di PT. Lihat bagian [P10] opsional.

---

## 2. Parameter X = 1

Semua dihitung dari Lampiran B.

| Parameter | Nilai |
|---|---|
| Blok HQ / Cabang / Gudang | `10.1.0.0/20` / `10.1.16.0/22` / `10.1.20.0/22` |
| Blok WAN / Loopback | `172.16.1.0/24` / `172.31.1.0/24` |
| Basis VLAN | `((1 mod 8)+1) × 100 = 200` |
| ISP-1 / ISP-2 | `203.0.113.0/29` / `198.51.100.0/29` (host `.1–.6`) |
| IPv6 | `2001:db8:1::/48` |
| Domain | `nusantara1.local` |
| `enable secret` | `Dmjk1#2026` |

### HQ (dialokasi besar → kecil)

| Segmen | Butuh | VLAN | Subnet | Mask | Gateway | Host | Broadcast |
|---|--:|--:|---|---|---|---|---|
| WiFi-Karyawan | 62 | 240 | `10.1.0.0/26` | 255.255.255.192 | 10.1.0.1 | .0.1–.0.62 | 10.1.0.63 |
| WiFi-Tamu | 31 | 250 | `10.1.0.64/26` | 255.255.255.192 | 10.1.0.65 | .0.65–.0.126 | 10.1.0.127 |
| HRD | 26 | 210 | `10.1.0.128/27` | 255.255.255.224 | 10.1.0.129 | .0.129–.0.158 | 10.1.0.159 |
| Keuangan | 13 | 220 | `10.1.0.160/28` | 255.255.255.240 | 10.1.0.161 | .0.161–.0.174 | 10.1.0.175 |
| Server | 9 | 230 | `10.1.0.176/28` | 255.255.255.240 | 10.1.0.177 | .0.177–.0.190 | 10.1.0.191 |
| Manajemen | 6 | 299 | `10.1.0.192/29` | 255.255.255.248 | 10.1.0.193 | .0.193–.0.198 | 10.1.0.199 |

Sisa HQ (pertumbuhan): `10.1.0.200 – 10.1.15.255` (3.896 alamat)

### Cabang

| Segmen | Butuh | VLAN | Subnet | Mask | Gateway | Host | Broadcast |
|---|--:|--:|---|---|---|---|---|
| Cabang-Umum | 31 | 280 | `10.1.16.0/26` | 255.255.255.192 | 10.1.16.1 | .16.1–.16.62 | 10.1.16.63 |
| WiFi-Cabang | 26 | 285 | `10.1.16.64/27` | 255.255.255.224 | 10.1.16.65 | .16.65–.16.94 | 10.1.16.95 |

Sisa: `10.1.16.96 – 10.1.19.255`

### Gudang

| Segmen | Butuh | VLAN | Subnet | Mask | Gateway | Host | Broadcast |
|---|--:|--:|---|---|---|---|---|
| Produksi | 41 | 260 | `10.1.20.0/26` | 255.255.255.192 | 10.1.20.1 | .20.1–.20.62 | 10.1.20.63 |
| WiFi-Gudang | 21 | 275 | `10.1.20.64/27` | 255.255.255.224 | 10.1.20.65 | .20.65–.20.94 | 10.1.20.95 |
| IoT | 16 | 270 | `10.1.20.96/27` | 255.255.255.224 | 10.1.20.97 | .20.97–.20.126 | 10.1.20.127 |

Sisa: `10.1.20.128 – 10.1.23.255` (896 alamat)

> VLAN **999 "PARKIR"** = tempat port tak terpakai (di-`shutdown`). VLAN 1 tidak dipakai.

### WAN `/30`

| Tautan | Prefix | Sisi A | Sisi B |
|---|---|---|---|
| HQ ↔ Cabang | `172.16.1.0/30` | R-HQ-1 `.1` | R-BR-1 `.2` |
| HQ ↔ Gudang | `172.16.1.4/30` | R-HQ-1 `.5` | R-GD-1 `.6` |
| HQ ↔ ISP-1 | `172.16.1.8/30` | R-HQ-1 `.9` | ISP-1 `.10` |
| HQ ↔ ISP-2 | `172.16.1.12/30` | R-HQ-1 `.13` | ISP-2 `.14` |

### Loopback, IPv6, publik, "Internet"

| Item | Nilai |
|---|---|
| Loopback | R-HQ-1 `172.31.1.1/32` · R-BR-1 `172.31.1.2/32` · R-GD-1 `172.31.1.3/32` |
| IPv6 | HRD `2001:db8:1:210::/64` · Keuangan `2001:db8:1:220::/64` |
| ISP-1 | PAT `203.0.113.1` · static NAT web `203.0.113.2` · cadangan `.3–.6` |
| ISP-2 | PAT `198.51.100.1` (hanya saat failover) · `.2–.6` cadangan |
| "Internet" simulasi `192.0.2.0/24` | ISP-1 Gi0/0 `.1` · ISP-2 Gi0/0 `.2` · SRV-CLOUD `.10` · PC-LUAR `.50` |

### Alamat statis segmen Server & Manajemen

| Perangkat | Alamat |
|---|---|
| SRV-DHCP-1 (DHCP + DNS) | `10.1.0.178` |
| SRV-WEB-1 | `10.1.0.179` |
| SRV-IOT-1 | `10.1.0.180` |
| (server) mask / gw / DNS | `255.255.255.240` / `10.1.0.177` / `10.1.0.178` |
| SW-HQ-1-01 / SW-HQ-1-02 | `10.1.0.194` / `10.1.0.195` |
| PC-ADM-1 | `10.1.0.196` (mask `.248`, gw `.193`) |
| Switch Cabang (SVI VLAN 280) | `10.1.16.2/26`, gw `10.1.16.1` |
| Switch Gudang (SVI VLAN 260) | `10.1.20.2/26`, gw `10.1.20.1` |

> Segmen Manajemen hanya ada di HQ. Switch remote dikelola lewat SVI di VLAN datanya; catat ini sebagai **selisih rencana-vs-kenyataan** di pekan 7.

---

## 3. Topologi & kabel

**R-HQ-1** (2911 + HWIC-2T)

| Port | Tersambung ke | Kabel |
|---|---|---|
| Gi0/0 | SW-HQ-1-01 Gi0/1 (trunk) | straight |
| Gi0/1 | R-BR-1 Gi0/1 | cross |
| Gi0/2 | R-GD-1 Gi0/1 | cross |
| Se0/0/0 | ISP-1 Se0/0/0 | serial (DCE di sisi ISP) |
| Se0/0/1 | ISP-2 Se0/0/0 | serial (DCE di sisi ISP) |

**Switch & perangkat akhir**

| Switch | Port | Perangkat | VLAN |
|---|---|---|--:|
| SW-HQ-1-01 | Gi0/1 | trunk ke R-HQ-1 | trunk |
| | Gi0/2 | trunk ke SW-HQ-1-02 Gi0/1 | trunk |
| | Fa0/1–3 | SRV-DHCP-1, SRV-WEB-1, SRV-IOT-1 | 230 |
| | Fa0/4 | PC-ADM-1 | 299 |
| | Fa0/5 | AP-HQ-1-01 (Staff) | 240 |
| | Fa0/6 | AP-HQ-1-02 (Guest) | 250 |
| | Fa0/7 | PC-HRD-1 | 210 |
| | Fa0/9, Fa0/10 | PC-KEU-1, PC-KEU-2 | 220 |
| SW-HQ-1-02 | Fa0/1 | PC-HRD-2 | 210 |
| SW-BR-1-01 | Gi0/1 | trunk ke R-BR-1 Gi0/0 | trunk |
| | Fa0/1 | AP-BR-1-01 | 285 |
| | Fa0/2–4 | PC-CAB | 280 |
| SW-GD-1-01 | Gi0/1 | trunk ke R-GD-1 Gi0/0 | trunk |
| | Fa0/1 | AP-GD-1-01 (WiFi-Gudang) | 275 |
| | Fa0/2 | AP-GD-1-02 (IoT) | 270 |
| | Fa0/3–5 | Produksi | 260 |
| | Fa0/6 | PC-IOT-TEST | 270 |

ISP-1 Gi0/0, ISP-2 Gi0/0, SRV-CLOUD, PC-LUAR → **SW-INET** (switch tanpa konfigurasi).

> Pekan 3–8: pasang PC uji di port AP (Fa0/5, Fa0/6, dst.) sebagai pengganti laptop wireless; ganti dengan AP saat pekan 9.

---

## 4. Lab pekan 1 dan 2 (sementara)

### [P1] Dua PC + satu switch 2960

PC1 `10.1.0.10/24`, PC2 `10.1.0.11/24`.

**Tantangan (satu baris MAC):** di switch jalankan `clear mac address-table dynamic`, lalu dari PC1 saja `ping 10.1.0.99` (tak ada pemiliknya). Hanya MAC PC1 yang dipelajari.

### [P2] R-HQ-1 sementara

Hapus sebelum P3 dengan `default interface g0/0` dan `default interface g0/1`.

```
interface gigabitEthernet0/0
 ip address 10.1.0.129 255.255.255.224
 ipv6 address 2001:DB8:1:210::1/64
 no shutdown
interface gigabitEthernet0/1
 ip address 10.1.0.161 255.255.255.240
 ipv6 address 2001:DB8:1:220::1/64
 no shutdown
ipv6 unicast-routing
```

| PC | IPv4 | IPv6 |
|---|---|---|
| PC-HRD-T | `10.1.0.130/27`, gw `10.1.0.129` | `2001:db8:1:210::10/64`, gw `::1` |
| PC-KEU-T | `10.1.0.162/28`, gw `10.1.0.161` | `2001:db8:1:220::10/64`, gw `::1` |

Uji: `ping 10.1.0.162` dan `ping 2001:db8:1:220::10` dari PC-HRD-T.

---

## 5. Router

### R-HQ-1

#### [DASAR]

```
enable
configure terminal
hostname R-HQ-1
no ip domain-lookup
enable secret Dmjk1#2026
end
```

#### [P3] Sub-interface (router-on-a-stick)

```
configure terminal
interface gigabitEthernet0/0
 description Trunk ke SW-HQ-1-01
 no shutdown
exit
interface gigabitEthernet0/0.210
 description Gateway HRD
 encapsulation dot1Q 210
 ip address 10.1.0.129 255.255.255.224
 ipv6 address 2001:DB8:1:210::1/64
exit
interface gigabitEthernet0/0.220
 description Gateway Keuangan
 encapsulation dot1Q 220
 ip address 10.1.0.161 255.255.255.240
 ipv6 address 2001:DB8:1:220::1/64
exit
interface gigabitEthernet0/0.230
 description Gateway Server
 encapsulation dot1Q 230
 ip address 10.1.0.177 255.255.255.240
exit
interface gigabitEthernet0/0.240
 description Gateway WiFi-Karyawan
 encapsulation dot1Q 240
 ip address 10.1.0.1 255.255.255.192
exit
interface gigabitEthernet0/0.250
 description Gateway WiFi-Tamu
 encapsulation dot1Q 250
 ip address 10.1.0.65 255.255.255.192
exit
interface gigabitEthernet0/0.299
 description Gateway Manajemen
 encapsulation dot1Q 299
 ip address 10.1.0.193 255.255.255.248
exit
ipv6 unicast-routing
end
write memory
```

#### [P4] WAN, loopback, rute

```
configure terminal
interface gigabitEthernet0/1
 description WAN ke R-BR-1
 ip address 172.16.1.1 255.255.255.252
 no shutdown
exit
interface gigabitEthernet0/2
 description WAN ke R-GD-1
 ip address 172.16.1.5 255.255.255.252
 no shutdown
exit
interface serial0/0/0
 description WAN ke ISP-1
 ip address 172.16.1.9 255.255.255.252
 no shutdown
exit
interface loopback0
 ip address 172.31.1.1 255.255.255.255
exit
ip route 10.1.16.0 255.255.252.0 172.16.1.2
ip route 10.1.20.0 255.255.252.0 172.16.1.6
ip route 172.31.1.2 255.255.255.255 172.16.1.2
ip route 172.31.1.3 255.255.255.255 172.16.1.6
ip route 0.0.0.0 0.0.0.0 172.16.1.10
end
write memory
```

#### [P5] DHCP relay + PAT

```
configure terminal
interface gigabitEthernet0/0.210
 ip helper-address 10.1.0.178
 ip nat inside
exit
interface gigabitEthernet0/0.220
 ip helper-address 10.1.0.178
 ip nat inside
exit
interface gigabitEthernet0/0.230
 ip nat inside
exit
interface gigabitEthernet0/0.240
 ip helper-address 10.1.0.178
 ip nat inside
exit
interface gigabitEthernet0/0.250
 ip helper-address 10.1.0.178
 ip nat inside
exit
interface gigabitEthernet0/0.299
 ip nat inside
exit
interface gigabitEthernet0/1
 ip nat inside
exit
interface gigabitEthernet0/2
 ip nat inside
exit
interface serial0/0/0
 ip nat outside
exit
access-list 10 permit 10.1.0.0 0.0.15.255
access-list 10 permit 10.1.16.0 0.0.7.255
ip nat pool POOL-ISP1 203.0.113.1 203.0.113.1 netmask 255.255.255.248
ip nat inside source list 10 pool POOL-ISP1 overload
end
write memory
```

> Relay **tidak** dipasang di `.230` dan `.299`: Server dan Manajemen tidak memakai DHCP. Tantangan reservasi: lihat [bagian 7](#tantangan-p5-reservasi-2-perangkat).

#### [P6] Hardening, ACL, SSH

```
configure terminal
service password-encryption
banner motd #Akses terbatas NusantaraNet. Hanya staf IT berwenang.#
ip domain-name nusantara1.local
username admin secret Dmjk1#2026
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
access-list 99 permit 10.1.0.192 0.0.0.7
line vty 0 4
 login local
 transport input ssh
 access-class 99 in
exit
!
ip access-list extended ACL-USER-IN
 permit udp any eq 68 any eq 67
 permit tcp any 10.1.0.176 0.0.0.15 eq 80
 permit udp any 10.1.0.176 0.0.0.15 eq 53
 permit tcp any 10.1.0.176 0.0.0.15 eq 53
 deny ip any 10.0.0.0 0.255.255.255
 deny ip any 172.16.0.0 0.15.255.255
 permit ip any any
exit
ip access-list extended ACL-TAMU-IN
 permit udp any eq 68 any eq 67
 permit udp any host 10.1.0.178 eq 53
 permit tcp any host 10.1.0.178 eq 53
 deny ip any 10.0.0.0 0.255.255.255
 deny ip any 172.16.0.0 0.15.255.255
 deny ip any 192.168.0.0 0.0.255.255
 permit ip any any
exit
interface gigabitEthernet0/0.210
 ip access-group ACL-USER-IN in
exit
interface gigabitEthernet0/0.220
 ip access-group ACL-USER-IN in
exit
interface gigabitEthernet0/0.240
 ip access-group ACL-USER-IN in
exit
interface gigabitEthernet0/0.250
 ip access-group ACL-TAMU-IN in
exit
end
write memory
```

> Segmen Manajemen (`.299`) sengaja **tanpa ACL** = akses penuh.

**Opsional — tutup kebocoran IPv6 HRD ↔ Keuangan** (cek dukungan di PT dulu):

```
ipv6 access-list V6-HRD-KEU
 deny ipv6 2001:DB8:1:210::/64 2001:DB8:1:220::/64
 deny ipv6 2001:DB8:1:220::/64 2001:DB8:1:210::/64
 permit ipv6 any any
interface gigabitEthernet0/0.210
 ipv6 traffic-filter V6-HRD-KEU in
interface gigabitEthernet0/0.220
 ipv6 traffic-filter V6-HRD-KEU in
```

#### [P10] ISP-2, failover, NAT route-map, port forwarding, ACL luar

```
configure terminal
interface serial0/0/1
 description WAN ke ISP-2
 ip address 172.16.1.13 255.255.255.252
 ip nat outside
 no shutdown
exit
ip route 0.0.0.0 0.0.0.0 172.16.1.14 10
!
no ip nat inside source list 10 pool POOL-ISP1 overload
ip nat pool POOL-ISP2 198.51.100.1 198.51.100.1 netmask 255.255.255.248
route-map RM-ISP1 permit 10
 match ip address 10
 match interface Serial0/0/0
exit
route-map RM-ISP2 permit 10
 match ip address 10
 match interface Serial0/0/1
exit
ip nat inside source route-map RM-ISP1 pool POOL-ISP1 overload
ip nat inside source route-map RM-ISP2 pool POOL-ISP2 overload
ip nat inside source static tcp 10.1.0.179 80 203.0.113.2 80
!
ip access-list extended ACL-OUTSIDE-IN
 deny ip 10.0.0.0 0.255.255.255 any
 permit icmp any any echo-reply
 permit icmp any any time-exceeded
 permit icmp any any unreachable
 permit tcp any any established
 permit tcp any host 203.0.113.2 eq 80
 deny ip any any
exit
interface serial0/0/0
 ip access-group ACL-OUTSIDE-IN in
exit
interface serial0/0/1
 ip access-group ACL-OUTSIDE-IN in
exit
end
write memory
```

**Fallback** bila route-map NAT ditolak PT:

```
no ip nat inside source route-map RM-ISP1 pool POOL-ISP1 overload
no ip nat inside source route-map RM-ISP2 pool POOL-ISP2 overload
ip nat inside source list 10 pool POOL-ISP1 overload
```

> Akibatnya saat failover translasi tetap memakai `203.0.113.1` dan internet **tidak** berfungsi lewat ISP-2. Tulis sebagai keterbatasan di laporan.

**Opsional — deteksi upstream** (kemungkinan tidak ada di PT; uji sendiri):

```
ip sla 1
 icmp-echo 192.0.2.10 source-interface Serial0/0/0
 frequency 5
ip sla schedule 1 life forever start-time now
track 1 ip sla 1 reachability
no ip route 0.0.0.0 0.0.0.0 172.16.1.10
ip route 0.0.0.0 0.0.0.0 172.16.1.10 track 1
```

---

### R-BR-1

#### [DASAR]

```
enable
configure terminal
hostname R-BR-1
no ip domain-lookup
enable secret Dmjk1#2026
end
```

#### [P3 + P4]

```
configure terminal
interface gigabitEthernet0/0
 description Trunk ke SW-BR-1-01
 no shutdown
exit
interface gigabitEthernet0/0.280
 description Gateway Cabang-Umum
 encapsulation dot1Q 280
 ip address 10.1.16.1 255.255.255.192
exit
interface gigabitEthernet0/0.285
 description Gateway WiFi-Cabang
 encapsulation dot1Q 285
 ip address 10.1.16.65 255.255.255.224
exit
interface gigabitEthernet0/1
 description WAN ke R-HQ-1
 ip address 172.16.1.2 255.255.255.252
 no shutdown
exit
interface loopback0
 ip address 172.31.1.2 255.255.255.255
exit
ip route 0.0.0.0 0.0.0.0 172.16.1.1
end
write memory
```

#### [P5] Relay (di router **lokal**, bukan di HQ)

```
configure terminal
interface gigabitEthernet0/0.280
 ip helper-address 10.1.0.178
exit
interface gigabitEthernet0/0.285
 ip helper-address 10.1.0.178
exit
end
write memory
```

#### [P6]

```
configure terminal
service password-encryption
banner motd #Akses terbatas NusantaraNet. Hanya staf IT berwenang.#
ip domain-name nusantara1.local
username admin secret Dmjk1#2026
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
access-list 99 permit 10.1.0.192 0.0.0.7
line vty 0 4
 login local
 transport input ssh
 access-class 99 in
exit
ip access-list extended ACL-USER-IN
 permit udp any eq 68 any eq 67
 permit tcp any 10.1.0.176 0.0.0.15 eq 80
 permit udp any 10.1.0.176 0.0.0.15 eq 53
 permit tcp any 10.1.0.176 0.0.0.15 eq 53
 deny ip any 10.0.0.0 0.255.255.255
 deny ip any 172.16.0.0 0.15.255.255
 permit ip any any
exit
interface gigabitEthernet0/0.280
 ip access-group ACL-USER-IN in
exit
interface gigabitEthernet0/0.285
 ip access-group ACL-USER-IN in
exit
end
write memory
```

---

### R-GD-1

#### [DASAR]

```
enable
configure terminal
hostname R-GD-1
no ip domain-lookup
enable secret Dmjk1#2026
end
```

#### [P3 + P4]

```
configure terminal
interface gigabitEthernet0/0
 description Trunk ke SW-GD-1-01
 no shutdown
exit
interface gigabitEthernet0/0.260
 description Gateway Produksi
 encapsulation dot1Q 260
 ip address 10.1.20.1 255.255.255.192
exit
interface gigabitEthernet0/0.275
 description Gateway WiFi-Gudang
 encapsulation dot1Q 275
 ip address 10.1.20.65 255.255.255.224
exit
interface gigabitEthernet0/0.270
 description Gateway IoT
 encapsulation dot1Q 270
 ip address 10.1.20.97 255.255.255.224
exit
interface gigabitEthernet0/1
 description WAN ke R-HQ-1
 ip address 172.16.1.6 255.255.255.252
 no shutdown
exit
interface loopback0
 ip address 172.31.1.3 255.255.255.255
exit
ip route 0.0.0.0 0.0.0.0 172.16.1.5
end
write memory
```

#### [P5]

```
configure terminal
interface gigabitEthernet0/0.260
 ip helper-address 10.1.0.178
exit
interface gigabitEthernet0/0.275
 ip helper-address 10.1.0.178
exit
interface gigabitEthernet0/0.270
 ip helper-address 10.1.0.178
exit
end
write memory
```

#### [P6] + [P9] Isolasi IoT

```
configure terminal
service password-encryption
banner motd #Akses terbatas NusantaraNet. Hanya staf IT berwenang.#
ip domain-name nusantara1.local
username admin secret Dmjk1#2026
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
access-list 99 permit 10.1.0.192 0.0.0.7
line vty 0 4
 login local
 transport input ssh
 access-class 99 in
exit
ip access-list extended ACL-USER-IN
 permit udp any eq 68 any eq 67
 permit tcp any 10.1.0.176 0.0.0.15 eq 80
 permit udp any 10.1.0.176 0.0.0.15 eq 53
 permit tcp any 10.1.0.176 0.0.0.15 eq 53
 deny ip any 10.0.0.0 0.255.255.255
 deny ip any 172.16.0.0 0.15.255.255
 permit ip any any
exit
ip access-list extended ACL-IOT-IN
 permit udp any eq 68 any eq 67
 permit udp any host 10.1.0.178 eq 53
 permit tcp any host 10.1.0.180 eq 80
 deny ip any any
exit
interface gigabitEthernet0/0.260
 ip access-group ACL-USER-IN in
exit
interface gigabitEthernet0/0.275
 ip access-group ACL-USER-IN in
exit
interface gigabitEthernet0/0.270
 ip access-group ACL-IOT-IN in
exit
end
write memory
```

> Bila sensor gagal mendaftar, lihat counter `show ip access-lists ACL-IOT-IN`. Kalau baris `deny` yang naik, ganti baris `tcp 80` menjadi `permit ip any host 10.1.0.180` (tetap hanya ke satu server).

---

### ISP-1

2911 + HWIC-2T.

```
enable
configure terminal
hostname ISP-1
interface serial0/0/0
 ip address 172.16.1.10 255.255.255.252
 clock rate 64000
 no shutdown
exit
interface gigabitEthernet0/0
 ip address 192.0.2.1 255.255.255.0
 no shutdown
exit
ip route 203.0.113.0 255.255.255.248 172.16.1.9
ip route 198.51.100.0 255.255.255.248 192.0.2.2
end
write memory
```

> `clock rate` hanya di ujung DCE kabel; kalau ditolak, pindahkan ke sisi HQ.

### ISP-2

2911 + HWIC-2T.

```
enable
configure terminal
hostname ISP-2
interface serial0/0/0
 ip address 172.16.1.14 255.255.255.252
 clock rate 64000
 no shutdown
exit
interface gigabitEthernet0/0
 ip address 192.0.2.2 255.255.255.0
 no shutdown
exit
ip route 198.51.100.0 255.255.255.248 172.16.1.13
end
write memory
```

---

## 6. Switch

Semua switch: **[P3]** VLAN + port, **[P6]** hardening + SSH (+ port-security di Gudang).

### Blok hardening [P6] (dipakai semua switch)

Ganti `vty 0 15` sesuai contoh; `access-list 99` sama di semua perangkat.

```
configure terminal
service password-encryption
banner motd #Akses terbatas NusantaraNet. Hanya staf IT berwenang.#
ip domain-name nusantara1.local
username admin secret Dmjk1#2026
crypto key generate rsa general-keys modulus 1024
ip ssh version 2
access-list 99 permit 10.1.0.192 0.0.0.7
line vty 0 15
 login local
 transport input ssh
 access-class 99 in
exit
end
write memory
```

### SW-HQ-1-01

#### [P3]

```
enable
configure terminal
hostname SW-HQ-1-01
no ip domain-lookup
enable secret Dmjk1#2026
vlan 210
 name HRD
vlan 220
 name KEUANGAN
vlan 230
 name SERVER
vlan 240
 name WIFI-KARYAWAN
vlan 250
 name WIFI-TAMU
vlan 299
 name MANAJEMEN
vlan 999
 name PARKIR
exit
interface range fastEthernet0/1-3
 switchport mode access
 switchport access vlan 230
exit
interface fastEthernet0/4
 switchport mode access
 switchport access vlan 299
exit
interface fastEthernet0/5
 switchport mode access
 switchport access vlan 240
exit
interface fastEthernet0/6
 switchport mode access
 switchport access vlan 250
exit
interface fastEthernet0/7
 switchport mode access
 switchport access vlan 210
exit
interface range fastEthernet0/9-10
 switchport mode access
 switchport access vlan 220
exit
interface fastEthernet0/8
 switchport mode access
 switchport access vlan 999
 shutdown
exit
interface range fastEthernet0/11-24
 switchport mode access
 switchport access vlan 999
 shutdown
exit
interface gigabitEthernet0/1
 description Trunk ke R-HQ-1
 switchport mode trunk
 switchport trunk allowed vlan 210,220,230,240,250,299
exit
interface gigabitEthernet0/2
 description Trunk ke SW-HQ-1-02
 switchport mode trunk
 switchport trunk allowed vlan 210,299
exit
interface vlan 299
 ip address 10.1.0.194 255.255.255.248
 no shutdown
exit
ip default-gateway 10.1.0.193
end
write memory
```

#### [P6]

Terapkan [blok hardening](#blok-hardening-p6-dipakai-semua-switch).

### SW-HQ-1-02

#### [P3]

```
enable
configure terminal
hostname SW-HQ-1-02
no ip domain-lookup
enable secret Dmjk1#2026
vlan 210
 name HRD
vlan 299
 name MANAJEMEN
vlan 999
 name PARKIR
exit
interface fastEthernet0/1
 switchport mode access
 switchport access vlan 210
exit
interface range fastEthernet0/2-24
 switchport mode access
 switchport access vlan 999
 shutdown
exit
interface gigabitEthernet0/1
 description Trunk ke SW-HQ-1-01
 switchport mode trunk
 switchport trunk allowed vlan 210,299
exit
interface gigabitEthernet0/2
 switchport access vlan 999
 shutdown
exit
interface vlan 299
 ip address 10.1.0.195 255.255.255.248
 no shutdown
exit
ip default-gateway 10.1.0.193
end
write memory
```

#### [P6]

Terapkan [blok hardening](#blok-hardening-p6-dipakai-semua-switch).

### SW-BR-1-01

#### [P3]

```
enable
configure terminal
hostname SW-BR-1-01
no ip domain-lookup
enable secret Dmjk1#2026
vlan 280
 name CABANG-UMUM
vlan 285
 name WIFI-CABANG
vlan 999
 name PARKIR
exit
interface fastEthernet0/1
 switchport mode access
 switchport access vlan 285
exit
interface range fastEthernet0/2-4
 switchport mode access
 switchport access vlan 280
exit
interface range fastEthernet0/5-24
 switchport mode access
 switchport access vlan 999
 shutdown
exit
interface gigabitEthernet0/1
 description Trunk ke R-BR-1
 switchport mode trunk
 switchport trunk allowed vlan 280,285
exit
interface gigabitEthernet0/2
 switchport access vlan 999
 shutdown
exit
interface vlan 280
 ip address 10.1.16.2 255.255.255.192
 no shutdown
exit
ip default-gateway 10.1.16.1
end
write memory
```

#### [P6]

Terapkan [blok hardening](#blok-hardening-p6-dipakai-semua-switch).

### SW-GD-1-01

#### [P3]

```
enable
configure terminal
hostname SW-GD-1-01
no ip domain-lookup
enable secret Dmjk1#2026
vlan 260
 name PRODUKSI
vlan 270
 name IOT
vlan 275
 name WIFI-GUDANG
vlan 999
 name PARKIR
exit
interface fastEthernet0/1
 switchport mode access
 switchport access vlan 275
exit
interface fastEthernet0/2
 switchport mode access
 switchport access vlan 270
exit
interface range fastEthernet0/3-5
 switchport mode access
 switchport access vlan 260
exit
interface fastEthernet0/6
 switchport mode access
 switchport access vlan 270
exit
interface range fastEthernet0/7-24
 switchport mode access
 switchport access vlan 999
 shutdown
exit
interface gigabitEthernet0/1
 description Trunk ke R-GD-1
 switchport mode trunk
 switchport trunk allowed vlan 260,270,275
exit
interface gigabitEthernet0/2
 switchport access vlan 999
 shutdown
exit
interface vlan 260
 ip address 10.1.20.2 255.255.255.192
 no shutdown
exit
ip default-gateway 10.1.20.1
end
write memory
```

#### [P6] Hardening + port-security Produksi

Terapkan [blok hardening](#blok-hardening-p6-dipakai-semua-switch), lalu:

```
configure terminal
interface range fastEthernet0/3-5
 switchport port-security
 switchport port-security maximum 1
 switchport port-security mac-address sticky
 switchport port-security violation restrict
exit
end
write memory
```

---

## 7. Server, PC, WLAN, IoT (GUI)

### SRV-DHCP-1 (Server-PT)

Alamat `10.1.0.178/28`, gw `10.1.0.177`, DNS `10.1.0.178`.

**Services → DHCP → On.** Hapus/nonaktifkan `serverPool`, buat pool berikut (DNS Server semua = `10.1.0.178`; *Start IP* melompati alamat yang dikecualikan):

| Pool | Gateway | Start IP | Mask | Max | Dikecualikan |
|---|---|---|---|--:|---|
| HQ-KARYAWAN | 10.1.0.1 | 10.1.0.6 | 255.255.255.192 | 57 | .1–.5 |
| HQ-TAMU | 10.1.0.65 | 10.1.0.70 | 255.255.255.192 | 57 | .65–.69 |
| HQ-HRD | 10.1.0.129 | 10.1.0.134 | 255.255.255.224 | 25 | .129–.133 |
| HQ-KEU | 10.1.0.161 | 10.1.0.164 | 255.255.255.240 | 11 | .161–.163 |
| BR-UMUM | 10.1.16.1 | 10.1.16.6 | 255.255.255.192 | 57 | .1–.5 |
| BR-WIFI | 10.1.16.65 | 10.1.16.70 | 255.255.255.224 | 25 | .65–.69 |
| GD-PROD | 10.1.20.1 | 10.1.20.6 | 255.255.255.192 | 57 | .1–.5 |
| GD-WIFI | 10.1.20.65 | 10.1.20.70 | 255.255.255.224 | 25 | .65–.69 |
| GD-IOT | 10.1.20.97 | 10.1.20.102 | 255.255.255.224 | 25 | .97–.101 |

**Services → DNS → On**, record A:

| Nama | Alamat |
|---|---|
| `www.nusantara1.local` | `10.1.0.179` |
| `erp.nusantara1.local` | `10.1.0.178` |
| `sensor.nusantara1.local` | `10.1.0.180` |

### Server lain

| Perangkat | Alamat | Pengaturan |
|---|---|---|
| SRV-WEB-1 | `10.1.0.179/28`, gw `.177` | HTTP On; ubah `index.html` jadi "NusantaraNet X=1" |
| SRV-IOT-1 | `10.1.0.180/28`, gw `.177` | Services → IoT On; buat akun (mis. `nusa1` / `iot-2026`) |
| SRV-CLOUD | `192.0.2.10/24`, gw `192.0.2.1` | HTTP On |
| PC-LUAR | `192.0.2.50/24`, gw `192.0.2.1` | — |

### PC (P3–P4: statis sementara; P5 dst: DHCP)

| PC | IPv4 sementara | IPv6 |
|---|---|---|
| PC-HRD-1 (SW-01 Fa0/7) | `10.1.0.130/27`, gw `.129` | `2001:db8:1:210::11/64`, gw `::1` |
| PC-HRD-2 (SW-02 Fa0/1) | `10.1.0.131/27`, gw `.129` | `2001:db8:1:210::12/64`, gw `::1` |
| PC-KEU-1 (Fa0/9) | `10.1.0.162/28`, gw `.161` | `2001:db8:1:220::11/64`, gw `::1` |
| PC-KEU-2 (Fa0/10) | `10.1.0.163/28`, gw `.161` | `2001:db8:1:220::12/64`, gw `::1` |
| PC-ADM-1 | `10.1.0.196/29`, gw `10.1.0.193`, DNS `10.1.0.178` | tetap statis |
| PC uji WiFi-Karyawan (port AP) | `10.1.0.3/26`, gw `.1` | — |
| PC uji WiFi-Tamu (port AP) | `10.1.0.67/26`, gw `.65` | — |
| PC-CAB-1 / PC-CAB-2 | `10.1.16.3` / `.4` `/26`, gw `10.1.16.1` | — |
| PC-PROD-1 / PC-PROD-2 | `10.1.20.3` / `.4` `/26`, gw `10.1.20.1` | — |

> Alamat sementara ini berada di rentang yang **dikecualikan** dari DHCP. Server tetap statis.

### Tantangan P5: reservasi 2 perangkat

Server-PT tidak bisa mengikat MAC ke alamat. Pilihan yang bisa diuji: di R-HQ-1, **hapus** `ip helper-address` pada `g0/0.210` (HRD dilayani router lokal):

```
ip dhcp excluded-address 10.1.0.129 10.1.0.133
ip dhcp pool RES-HRD1
 host 10.1.0.134 255.255.255.224
 hardware-address <MAC PC-HRD-1>
 default-router 10.1.0.129
 dns-server 10.1.0.178
```

Ulangi sebagai `RES-HRD2` untuk `.135`. Verifikasi: `show ip dhcp binding`. Cek dukungan `hardware-address` di PT (Prompt B), dan catat bahwa HRD kini menyimpang dari "DHCP terpusat" sebagai trade-off.

**Mengapa lebih baik daripada statis di klien:** alamat tetap dikelola di satu tempat, klien tetap DHCP (gateway/DNS ikut berubah otomatis), dan catatan siapa-memakai-apa tetap terpusat.

### WLAN (P9)

AP-PT: **Config → Port 1** (SSID, Authentication WPA2-PSK, Channel).

| AP | Lokasi | SSID | Port switch | VLAN | Kanal | Password |
|---|---|---|---|--:|--:|---|
| AP-HQ-1-01 | HQ | NusantaraNet-Staff | SW-HQ-1-01 Fa0/5 | 240 | 1 | `Staff#2026Nusa1` |
| AP-HQ-1-02 | HQ | NusantaraNet-Guest | SW-HQ-1-01 Fa0/6 | 250 | 6 | `Guest#2026Nusa1` |
| AP-BR-1-01 | Cabang | NusantaraNet-Staff | SW-BR-1-01 Fa0/1 | 285 | 11 | `Staff#2026Nusa1` |
| AP-GD-1-01 | Gudang | NusantaraNet-Gudang | SW-GD-1-01 Fa0/1 | 275 | 1 | `Gudang#2026Nusa1` |
| AP-GD-1-02 | Gudang | NusantaraNet-IoT | SW-GD-1-01 Fa0/2 | 270 | 11 | `IoT#2026Nusa1` |

**Laptop:** power off → ganti modul **WPC300N** → power on → Config → Wireless0 (SSID, WPA2-PSK, passphrase, IP = DHCP). Laptop uji: LAP-KAR-1, LAP-TAMU-1, LAP-WCAB-1, LAP-WGD-1.

> SSID yang sama di dua lokasi (Staff di HQ = VLAN 240, di Cabang = VLAN 285) sah: SSID hanyalah nama, yang menentukan segmen adalah VLAN pada port AP.

### IoT

Perangkat tersambung ke AP IoT (SSID `NusantaraNet-IoT`), alamat DHCP.

| Perangkat | Jenis |
|---|---|
| SENSOR-GD-1 | Temperature Monitor |
| SENSOR-GD-2 | Humidity / Motion |
| AKT-GD-1 | Fan |

Tiap perangkat: **Config → IoT Server → Remote Server**, alamat `10.1.0.180` (atau `sensor.nusantara1.local`), user `nusa1`.

Di SRV-IOT-1 → **IoT → Conditions**: *Temperature ≥ 30 → Fan On*.
