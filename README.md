# Laporan Tugas 1 Jaringan Komputer - Subnetting & Routing

**Nama:** (Isi Nama Anda)
**NRP:** 5027241084
**Kelas:** (Isi Kelas Anda)

---

## 1. Desain Topologi
Topologi yang digunakan adalah **Hub-and-Spoke** yang menghubungkan Kantor Pusat (5 LAN) dan Kantor Cabang (1 LAN).

<img width="1652" height="715" alt="image" src="https://github.com/user-attachments/assets/825846fd-1d3d-455b-8990-04ef65ffcb7f" />

---

## 2. Perhitungan VLSM (Variable Length Subnet Mask)

**Base Network:** 10.124.0.0/8 (Hasil dari NRP mod 256 = 124)

| Unit Kerja | Kebutuhan Host | Prefix | Subnet Mask | Network Address | Gateway (Router) |
| :--- | :--- | :--- | :--- | :--- | :--- |
| Sekretariat | 380 | /23 | 255.255.254.0 | 10.124.0.0 | 10.124.0.1 |
| Kurikulum | 220 | /24 | 255.255.255.0 | 10.124.2.0 | 10.124.2.1 |
| Guru & Tendik | 95 | /25 | 255.255.255.128 | 10.124.3.0 | 10.124.3.1 |
| Sarpras | 45 | /26 | 255.255.255.192 | 10.124.3.128 | 10.124.3.129 |
| Pengawas (Cabang) | 18 | /27 | 255.255.255.224 | 10.124.3.192 | 10.124.3.193 |
| Server & Admin | 6 | /29 | 255.255.255.248 | 10.124.3.224 | 10.124.3.225 |
| WAN Link | 2 | /30 | 255.255.255.252 | 10.124.3.232 | 10.124.3.233 |

---

## 3. CIDR & Supernetting
Seluruh jaringan di Kantor Pusat (Sekretariat s.d. Server) digabungkan (agregasi) menjadi satu blok besar agar tabel routing di Cabang lebih efisien.

* **Range Awal:** 10.124.0.0
* **Range Akhir:** 10.124.3.255
* **Hasil CIDR:** `10.124.0.0/22`

---

## 4. Konfigurasi Router (Script)

### Router Pusat (R-Pusat)
Interface menggunakan VLAN (SVI) karena modul switch NIM-ES2-4.

```bash
! Konfigurasi Port Fisik Layer 3
interface G0/0/0
 ip address 10.124.0.1 255.255.254.0
interface G0/0/1
 ip address 10.124.2.1 255.255.255.0

! Konfigurasi VLAN Layer 2
vlan 30
 name Guru
vlan 40
 name Sarpras
vlan 50
 name Server

! Interface VLAN (Gateway)
interface vlan 30
 ip address 10.124.3.1 255.255.255.128
interface vlan 40
 ip address 10.124.3.129 255.255.255.192
interface vlan 50
 ip address 10.124.3.225 255.255.255.248

! Link WAN
interface Serial0/1/0
 ip address 10.124.3.233 255.255.255.252

! Static Route ke Cabang
ip route 10.124.3.192 255.255.255.224 10.124.3.234

```

### Router Cabang (R-Cabang)

```bash
! LAN Pengawas
interface G0/0/0
 ip address 10.124.3.193 255.255.255.224

! Link WAN
interface Serial0/1/0
 ip address 10.124.3.234 255.255.255.252

! Routing CIDR (Supernet) ke Pusat
ip route 10.124.0.0 255.255.252.0 10.124.3.233
```
