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