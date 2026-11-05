
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
