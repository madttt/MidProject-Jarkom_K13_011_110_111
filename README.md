# Mid-Project Komunikasi Data dan Jaringan Komputer
# Implementasi Jaringan Smart City Kota Surabaya


---

## 1. Gambar Topologi


## 2. Alasan Pemilihan Skema Pengalamatan

---

## 3. Penjelasan Desain Topologi

### 3.1 Pemilihan Perangkat

| Perangkat | Dipakai untuk | Alasan |
|---|---|---|
| Router 2911 | R-ISP, R-DC, R-BARAT, R-TIMUR, R-SELATAN, R-PELABUHAN | Memiliki 3 port GigabitEthernet bawaan. Kebutuhan keenam router ini paling banyak 3 port, sehingga tidak perlu modul tambahan. |
| Router-PT-Empty + modul PT-ROUTER-NM-1CGE | R-PUSAT dan R-UTARA | R-PUSAT membutuhkan 8 port dan R-UTARA 4 port, melebihi 3 port bawaan 2911. Router-PT-Empty dapat ditambah modul port Gigabit sesuai kebutuhan. |
| Switch 2960 | Semua LAN | Satu LAN berisi banyak host, sehingga perlu satu titik kumpul. |
| Server-PT | Server Balai Kota, Server dan Storage DC, Server 8.8.8.8 | Perlu menjalankan layanan (Web, DNS, DHCP, FTP, Email) pada tab Services. |
| PC-PT | Pegawai, sensor, CCTV, PJU, SCADA, kantor | Mewakili perangkat pengguna atau perangkat IoT yang meminta IP dinamis. |
| Access Point-PT | WiFi Publik | Menyediakan akses nirkabel bagi warga. |
| Smartphone-PT | Klien WiFi Publik | Memiliki kartu jaringan wireless bawaan. PC-PT biasa hanya mendukung kabel. |

### 3.2 Alasan Penggunaan Switch di Setiap LAN

1. **Satu LAN berisi banyak host.** Satu port router hanya menerima satu kabel, sehingga tanpa switch hanya satu perangkat yang dapat terhubung.
2. **Hemat port router.** Satu LAN memakai satu port router berapa pun jumlah perangkatnya. R-PUSAT sudah memakai 8 port, sehingga tidak ada ruang untuk satu port per perangkat.
3. **Mudah dikembangkan.** Perangkat baru cukup dicolokkan ke switch tanpa mengubah router atau subnet.
4. **Konsisten.** Semua LAN memakai pola yang sama sehingga topologi seragam.
5. **Kabel sederhana.** Router ke switch dan switch ke perangkat memakai kabel Straight-Through.

Setiap LAN memakai **switch terpisah** karena tiap LAN adalah subnet yang berbeda. Memakai satu switch bersama membutuhkan VLAN, yang tidak diperlukan pada soal ini dan menambah kompleksitas.

### 3.3 Media Link: Kabel atau Wireless

| Lokasi | Media | Alasan |
|---|---|---|
| Backbone antar-router (L1 sampai L6, R-ISP ke R-PUSAT) | Kabel | Jalur utama harus stabil dan berkapasitas tinggi, serta mengikuti Gambar 1. |
| Server (Balai Kota, DC, Internet) | Kabel | Server menjalankan layanan penting dan harus selalu tersedia. |
| CCTV Kota | Kabel | Rekaman video membutuhkan bandwidth besar dan koneksi tanpa putus ke server FTP. |
| Sensor (lalu lintas, banjir, SCADA) dan PJU Pintar | Kabel | Perangkat tetap yang data pantauannya tidak boleh hilang oleh gangguan sinyal. |
| Pegawai dan Kantor (Balai Kota, Pelabuhan) | Kabel | Perangkat kerja tetap, butuh koneksi stabil untuk email dan portal. |
| Puskesmas dan Sekolah | Kabel | Layanan publik tetap yang membutuhkan koneksi andal. |
| **WiFi Publik** (Utara, Timur, Selatan, Barat) | **Wireless** | Dipakai warga dengan perangkat bergerak. Access Point dihubungkan ke switch dengan kabel agar jalur ke router tetap stabil. |

### 3.4 Jenis Kabel

| Sambungan | Kabel |
|---|---|
| Router ke Router (backbone) | Copper Cross-Over |
| Router ke Server Internet (8.8.8.8) | Copper Cross-Over |
| Router ke Switch | Copper Straight-Through |
| Switch ke PC, Server, atau Access Point | Copper Straight-Through |
| Access Point ke Smartphone | Wireless (SSID yang sama) |

### 3.5 Penambahan Port pada Router

| Router | Link ke router lain | LAN | Total port | Solusi |
|---|---|---|---|---|
| R-ISP | 1 (ke R-PUSAT) | 1 (Server 8.8.8.8) | 2 | Port bawaan 2911 |
| R-PUSAT | 6 (R-ISP, L1 sampai L5) | 2 (Pegawai, Server Balai Kota) | **8** | Router-PT-Empty + 8 modul NM-1CGE |
| R-DC | 1 | 2 | 3 | Port bawaan 2911 |
| R-BARAT | 1 | 2 | 3 | Port bawaan 2911 |
| R-UTARA | 2 (L2, L6) | 2 | **4** | Router-PT-Empty + 4 modul NM-1CGE |
| R-TIMUR | 1 | 2 | 3 | Port bawaan 2911 |
| R-SELATAN | 1 | 2 | 3 | Port bawaan 2911 |
| R-PELABUHAN | 1 | 2 | 3 | Port bawaan 2911 |

### 3.5.1 Pemetaan Port

**R-PUSAT**

| Port | Tujuan |
|---|---|
| G0/0 | R-ISP |
| G1/0 | L1 ke R-DC |
| G2/0 | L2 ke R-UTARA |
| G3/0 | L3 ke R-SELATAN |
| G4/0 | L4 ke R-TIMUR |
| G5/0 | L5 ke R-BARAT |
| G6/0 | LAN Pegawai Balai Kota |
| G7/0 | LAN Server Balai Kota |

**R-UTARA**

| Port | Tujuan |
|---|---|
| G0/0 | L2 ke R-PUSAT |
| G1/0 | L6 ke R-PELABUHAN |
| G2/0 | LAN WiFi Publik Utara |
| G3/0 | LAN Sensor Lalu Lintas Utara |

**Router 2911**

| Router | G0/0 | G0/1 | G0/2 |
|---|---|---|---|
| R-ISP | ke R-PUSAT | Server 8.8.8.8 | tidak dipakai |
| R-DC | L1 ke R-PUSAT | LAN CCTV Kota | LAN Server dan Storage DC |
| R-BARAT | L5 ke R-PUSAT | LAN WiFi Barat | LAN Puskesmas dan Sekolah |
| R-TIMUR | L4 ke R-PUSAT | LAN WiFi Timur | LAN Sensor Banjir |
| R-SELATAN | L3 ke R-PUSAT | LAN WiFi Selatan | LAN PJU Pintar |
| R-PELABUHAN | L6 ke R-UTARA | LAN Sensor SCADA | LAN Kantor Pelabuhan |

---

## 4. Label LAN


| LAN | Network ID/prefix | Jumlah host |
|---|---|---|
| ... | ... | ... |

## Soal 3 
### Penjelasan singkat
Membuat semua LAN Smart City Surabaya saling terhubung menggunakan **static routing**: R-PUSAT memiliki static route ke semua LAN cabang, router cabang memiliki default route ke R-PUSAT, dan R-UTARA memiliki static route ke LAN R-PELABUHAN. R-ISP sengaja **tidak** diberi route ke `10.104.0.0/16` (disiapkan untuk Soal 5 NAT). Output file baru: `v2_static.pkt` (file asli `v1.pkt` tidak diubah).

### Proses pengerjaan
1. Ganti `SW-PEGAWAI-BALAIKOTA` dari router 2911 ke switch 2960; hubungkan `R-PUSAT Gi6/0 ↔ SW-PEGAWAI Fa0/1`, `PC-PEGAWAI Fa0 ↔ SW-PEGAWAI Fa0/2`, dan `SRV-DC Fa0 ↔ SW-DC Fa0/2`.
2. Set IP static: `PC-PEGAWAI = 10.104.224.10/21` (gw `10.104.224.1`), `SRV-DC = 10.104.239.10/24`, `PC-CCTV = 10.104.96.10/19`.
3. Tambahkan static route di R-PUSAT ke setiap LAN cabang (contoh: `10.104.96.0/19 via 10.104.241.34`, `10.104.239.0/24 via 10.104.241.34`, `10.104.236.0/23 via 10.104.241.38`, `10.104.238.0/24 via 10.104.241.50`).
4. Tambahkan default route di setiap router cabang ke R-PUSAT (R-DC via `10.104.241.33`, R-UTARA via `10.104.241.37`, dst.).
5. Tambahkan static route R-UTARA → Pelabuhan: `10.104.160.0/19` dan `10.104.240.0/25 via 10.104.241.54`.
6. Jangan tambahkan route internal di R-ISP.
7. Perbaiki binding kabel agar endpoint tidak tertimpa port switch (urutan endpoint dibalik).

### Verifikasi
- `PC-PEGAWAI`: `ipconfig` → IP `10.104.224.10/21`, gw `10.104.224.1`.
- `ping 10.104.224.1` → 4/4 reply.
- `ping 10.104.96.10` (CCTV) → reply (percobaan kedua, ARP/STP warm-up).
- R-PUSAT: `show ip route` → static route ke LAN cabang + default `0.0.0.0/0 via 203.0.113.54`.
- Router cabang: `show ip route` → default route via R-PUSAT; R-UTARA juga route ke `10.104.160.0/19` & `10.104.240.0/25`.
- R-ISP: `show ip route` → **tidak** ada route ke `10.104.0.0/16`.
- Workspace List → Links = 31, Devices = 39; tidak ada `Router0/Switch1/PC0`.
## Soal 4 
### Penjelasan singkat
Seluruh end device (pengguna) diubah dari IP static menjadi **DHCP** (IP, mask, gateway, DNS otomatis). Interface router dan server tetap static. Subnet CCTV (`10.104.96.0/19`) mendapat DHCP dari `SRV-DC` (pool `CCTV-KOTA`) via `ip helper-address 10.104.239.10` pada `R-DC Gi0/1`, bukan dari R-DC. Dibuat 11 pool router + 1 pool CCTV. DNS semua pool = `10.104.239.10`. Output: `v4_dhcp.pkt`.
### Proses pengerjaan
1. Buat 11 pool DHCP di router (PEGAWAI, WIFI-UTARA, SENSOR-LALIN, WIFI-TIMUR, SENSOR-BANJIR, WIFI-BARAT, PUSKESMAS, WIFI-SELATAN, PJU-SELATAN, SCADA-PELABUHAN, KANTOR-PELABUHAN) dengan network/mask/gateway sesuai VLSM dan `dns-server 10.104.239.10`, `excluded-address` = gateway.
3. Buat pool `CCTV-KOTA` di `SRV-DC` (Services → DHCP): network `10.104.96.0/19`, gw `10.104.96.1`, DNS `10.104.239.10`, start `10.104.96.2`.
4. Set `ip helper-address 10.104.239.10` pada `R-DC Gi0/1`; pastikan R-DC tidak punya pool CCTV.
5. Set 12 end device ke `DHCP=true` dan kosongkan field IP static.
6. Pastikan router/server tetap static.
### Verifikasi
- R-PUSAT: `show running-config` → `ip dhcp pool PEGAWAI` + `dns-server 10.104.239.10`.
- SRV-DC → Services → DHCP → pool `CCTV-KOTA` (network `10.104.96.0/19`, gw `10.104.96.1`, DNS `10.104.239.10`).
- R-DC: `show running-config interface GigabitEthernet0/1` → `ip helper-address 10.104.239.10`.
- `PC-CCTV-KOTA`: `ipconfig /all` → IP `10.104.96.2`, gw `10.104.96.1`, DHCP server `10.104.239.10`, DNS `10.104.239.10`.
- HP lease: UTARA `10.104.0.2`, TIMUR `10.104.128.2`, SELATAN `10.104.64.2`, BARAT `10.104.192.3`.
- `nslookup www.k13.smartcity.lab` dan `ping www.k13.smartcity.lab` dari PC DHCP → reply.
- Link = 31, device = 39; server/router static tetap.
## Soal 5 
### Penjelasan singkat
Seluruh host internal (`10.104.0.0/16`) mengakses internet lewat **satu IP publik** milik R-PUSAT menggunakan **NAT overload (PAT)**. R-ISP tetap tanpa route ke internal. Output: `v5_nat.pkt`.

### Proses pengerjaan
1. Di R-PUSAT:
   - `Gi0/0` (IP publik `203.0.113.53/30` ke R-ISP) → `ip nat outside`.
   - `Gi1/0`–`Gi7/0` (link internal + 2 LAN Balai Kota) → `ip nat inside`.
   - `access-list 1 permit 10.104.0.0 0.0.255.255`.
   - `ip nat inside source list 1 interface GigabitEthernet0/0 overload`.
3. Biarkan R-ISP tanpa static route ke internal (hanya tahu `203.0.113.52/30` dan `8.8.8.0/24`).
4. `SRV-INTERNET = 8.8.8.8/24`, gw `8.8.8.1`; R-ISP `Gi0/1 = 8.8.8.1/24`.

### Verifikasi
- R-PUSAT: `show running-config` → `ip nat outside` (Gi0/0), `ip nat inside` (Gi1/0–Gi7/0), ACL `10.104.0.0 0.0.255.255`, `ip nat inside source list 1 interface GigabitEthernet0/0 overload`.
- R-ISP: `show ip route` → tidak ada route ke `10.104.0.0/16`.
- `PC-PEGAWAI`: `ping 8.8.8.8` → `Reply from 8.8.8.8` (ping pertama boleh timeout karena ARP warm-up, ulangi).
- R-PUSAT: `show ip nat statistics` → Outside `Gi0/0`, Inside 7 interface, `Hits > 0`, `Expired translations > 0`.
- DHCP/layanan/routing internal tetap berfungsi.
## Soal 6 
### Penjelasan singkat
Membuat **GRE tunnel** `Tunnel0` antara R-PUSAT (Balai Kota) dan R-PELABUHAN, sehingga lalu lintas antara kedua area harus lewat tunnel (menggantikan jalur via R-UTARA). Dilengkapi dokumentasi: mengapa tunnel vs jalur biasa, dan mengapa GRE tidak mengenkripsi (bonus IPsec). Output: `v6_gre.pkt`.

### Proses pengerjaan
1. Upgrade IOS R-PUSAT dari `pt_12.2` ke `2900_universal_base_15.1` agar mendukung `interface Tunnel` (port & IP tetap).
3. Konfigurasi GRE tunnel:
   - R-PUSAT: `Tunnel0` IP `192.168.0.1/30`, source `Gi2/0` (`10.104.241.37`), dest `10.104.241.54`.
   - R-PELABUHAN: `Tunnel0` IP `192.168.0.2/30`, source `Gi0/0` (`10.104.241.54`), dest `10.104.241.37`.
4. Route lewat tunnel:
   - R-PUSAT: `10.104.160.0/19` dan `10.104.240.0/25 via 192.168.0.2`.
   - R-PELABUHAN: `10.104.224.0/21` dan `10.104.241.0/27 via 192.168.0.1`.
5. Dokumentasi alasan: tunnel membuat link point-to-point logis langsung sehingga traffic Balai Kota ↔ Pelabuhan tidak bergantung routing R-UTARA; GRE hanya encapsulation (bukan encryption), payload tetap plaintext → perlu IPsec untuk kerahasiaan.

### Verifikasi
- R-PUSAT: `show ip interface brief` → `Tunnel0` up/up, IP `192.168.0.1`.
- R-PUSAT: `ping 192.168.0.2` → 5/5 (percobaan pertama boleh 2/5, ulangi).
- R-PUSAT: `traceroute 10.104.240.1` → hop pertama `192.168.0.2` (tunnel, bukan R-UTARA).
- `PC-PEGAWAI` (Balai Kota): `ping 10.104.240.1` dan `ping 10.104.160.1` → reply; sebaliknya dari Pelabuhan ke Balai Kota juga berhasil.
- R-PUSAT & R-PELABUHAN: `show running-config` → cek Tunnel0 + route via `192.168.0.x`.
- DHCP/NAT/layanan/routing internal tetap berfungsi; link = 31, device = 39.
