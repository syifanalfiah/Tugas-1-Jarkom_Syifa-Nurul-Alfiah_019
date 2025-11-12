## Struktur Jaringan

### Kantor Pusat
| Ruang / Bidang             | Jumlah Host Aktif |
|-----------------------------|------------------:|
| Sekretariat                 | 380 host |
| Bidang Kurikulum            | 220 host |
| Bidang Guru & Tendik        | 95 host |
| Bidang Sarana Prasarana     | 45 host |
| Server & Admin              | 6 host |

### Kantor Cabang
| Ruang / Bidang             | Jumlah Host Aktif |
|-----------------------------|------------------:|
| Bidang Pengawas Sekolah (Cabang) | 18 host |

---

## Base Network
Setiap mahasiswa menggunakan **base network unik** berdasarkan rumus berikut:

```

10.(NRP mod 256).0.0

```

**NRP Syifa:** → `5027241019`

---

## Perhitungan Subnetting (VLSM)

| No | Nama Subnet | Jumlah Host | Prefix | Network | Netmask | Range Host | Broadcast | Gateway |
|----|--------------|-------------|--------|----------|----------|-------------|------------|----------|
| 1 | Sekretariat | 380 | /23 | 10.59.0.0 | 255.255.254.0 | 10.59.0.1 – 10.59.1.254 | 10.59.1.255 | 10.59.0.1 |
| 2 | Kurikulum | 220 | /24 | 10.59.2.0 | 255.255.255.0 | 10.59.2.1 – 10.59.2.254 | 10.59.2.255 | 10.59.2.1 |
| 3 | Guru & Tendik | 95 | /25 | 10.59.3.0 | 255.255.255.128 | 10.59.3.1 – 10.59.3.126 | 10.59.3.127 | 10.59.3.1 |
| 4 | Sarana Prasarana | 45 | /26 | 10.59.3.128 | 255.255.255.192 | 10.59.3.129 – 10.59.3.190 | 10.59.3.191 | 10.59.3.129 |
| 5 | Server & Admin | 6 | /29 | 10.59.3.192 | 255.255.255.248 | 10.59.3.193 – 10.59.3.198 | 10.59.3.199 | 10.59.3.193 |
| 6 | Pengawas (Cabang) | 18 | /27 | 10.59.3.200 | 255.255.255.224 | 10.59.3.201 – 10.59.3.222 | 10.59.3.223 | 10.59.3.201 |
| 7 | Link Antar Router | 2 | /30 | 10.59.3.224 | 255.255.255.252 | 10.59.3.225 – 10.59.3.226 | 10.59.3.227 | 10.59.3.225 |

---

## Hasil CIDR (Supernetting)

Dari hasil subnetting di atas, seluruh subnet di kantor pusat dapat digabung dengan CIDR menjadi satu supernet:

```

10.59.0.0/20

````

| CIDR | Network | Range Host | Broadcast |
|-------|----------|-------------|------------|
| /20 | 10.59.0.0 | 10.59.0.1 – 10.59.15.254 | 10.59.15.255 |

Supernetting digunakan untuk menyederhanakan routing antar kantor pusat dan cabang.

---

## 🖥️ Topologi Jaringan

Topologi dirancang menggunakan **Cisco Packet Tracer** dengan komponen utama sebagai berikut:
- **1 Router Pusat (Cisco 2911)**
- **1 Router Cabang (Cisco 2911)**
- **1 Core Switch (2950-24 / 2960)**
- **5 Switch Akses (Sekretariat, Kurikulum, Guru & Tendik, SarPras, Server/Admin)**
- **1 Switch di Kantor Cabang**
- **Host-PC di setiap VLAN**
- **Link antar router menggunakan kabel serial (DCE/DTE)**

### Deskripsi Koneksi
- **Router Pusat ↔ Core Switch** (GigabitEthernet0/1) — Trunk
- **Core Switch ↔ Switch Unit Kerja** (FastEthernet0/x) — Trunk per VLAN
- **Router Pusat ↔ Router Cabang** — Serial (point-to-point link)
- **Switch Cabang ↔ Host Pengawas** — VLAN 60

---

## Konfigurasi Trunking (Contoh Core-Switch)

```bash
enable
configure terminal
interface gigabitEthernet0/1
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
 no shutdown
exit
interface range fastEthernet0/2 - 0/6
 switchport mode trunk
 switchport trunk allowed vlan 10,20,30,40,50,60
 no shutdown
end
write memory
````

---

## VLAN Mapping

| VLAN ID | Nama VLAN   | Bidang / Ruang           |
| ------- | ----------- | ------------------------ |
| 10      | Sekretariat | Sekretariat              |
| 20      | Kurikulum   | Bidang Kurikulum         |
| 30      | GuruTendik  | Bidang Guru & Tendik     |
| 40      | Sarpras     | Bidang Sarana Prasarana  |
| 50      | ServerAdmin | Server & Admin           |
| 60      | Pengawas    | Bidang Pengawas (Cabang) |

---

## Routing antar Kantor

**Static Routing / RIP** digunakan agar jaringan antar kantor saling terhubung.

### Router Pusat

```bash
enable
configure terminal
router rip
 version 2
 network 10.59.0.0
 no auto-summary
exit
write memory
```

### Router Cabang

```bash
enable
configure terminal
router rip
 version 2
 network 10.59.0.0
 no auto-summary
exit
write memory
```

<img width="1345" height="465" alt="image" src="https://github.com/user-attachments/assets/570b5bd6-b09b-4a02-a4f0-bcfd561bd863" />

<img width="422" height="260" alt="image" src="https://github.com/user-attachments/assets/0a68cfc0-c6d5-4984-beaf-2990727ca468" />

