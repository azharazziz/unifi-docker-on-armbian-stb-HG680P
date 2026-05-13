# UniFi Network on Armbian (ARM64) via Docker

Deploy UniFi Network Application di Armbian ARM64 menggunakan Docker + MongoDB.

---

## Features

- UniFi Network Application (LinuxServer image)
- MongoDB 4.0 pada image `arm64v8/mongo:4.0` untuk ARM64
- Volume persisten (`./config`, `./mongo`)
- Restart otomatis
- Port mapping untuk UI, adoption, dan STUN

---

## Requirements

- Armbian / Debian-based OS (ARM64)
- Docker
- Docker Compose
- Minimal 2GB RAM (disarankan 4GB)

Install Docker:

```bash
curl -fsSL https://get.docker.com | sh
```

Install Docker Compose:

```bash
apt install docker-compose-plugin -y
```

---

## Folder structure

```text
unifi/
├── docker-compose.yml
├── config/          # data UniFi (dibuat setelah container jalan)
└── mongo/           # data MongoDB
```

---

## Installation

### 1. Siapkan folder dan berkas

```bash
mkdir -p unifi && cd unifi
```

Salin `docker-compose.yml` dari repositori ini ke folder tersebut (atau clone repo ini lalu `cd` ke dalamnya).

---

### 2. Buat user di MongoDB (wajib untuk pertama kali)

Image **linuxserver/unifi-network-application** menghubungkan ke MongoDB dengan kredensial dari environment (`MONGO_USER`, `MONGO_PASS`, `MONGO_DBNAME`). User tersebut **tidak** dibuat otomatis di dalam database; Anda harus membuatnya sekali di MongoDB agar UniFi bisa autentikasi.

Pastikan nilai di bawah **sama persis** dengan yang ada di `docker-compose.yml` Anda. Contoh default di compose repo ini:

- `MONGO_USER` → `unifi`
- `MONGO_PASS` → `unifi123`
- `MONGO_DBNAME` → `unifi`

Langkahnya:

1. Jalankan hanya layanan MongoDB:

   ```bash
   docker compose up -d mongo
   ```

2. Tunggu sekitar 10–30 detik sampai MongoDB siap menerima koneksi.

3. Buat user dengan role `readWrite` pada database `unifi` (satu baris, untuk shell bash di Armbian):

   ```bash
   docker exec -it unifi-mongo mongo admin --eval 'db.getSiblingDB("unifi").createUser({ user: "unifi", pwd: "unifi123", roles: [{ role: "readWrite", db: "unifi" }] })'
   ```

   Jika Anda mengubah user, password, atau nama database di compose, ganti string di dalam `createUser` agar cocok dengan `MONGO_USER`, `MONGO_PASS`, dan `MONGO_DBNAME`.

4. Keluaran sukses biasanya berisi `"ok" : 1`. Jika muncul error user sudah ada (`code 11000` / "already exists"), user sudah pernah dibuat; untuk volume Mongo baru biasanya tidak terjadi.

**Cara interaktif (opsional):**

```bash
docker exec -it unifi-mongo mongo
```

Di prompt `mongo`:

```javascript
use unifi
db.createUser({
  user: "unifi",
  pwd: "unifi123",
  roles: [{ role: "readWrite", db: "unifi" }]
})
```

Ketik `exit` untuk keluar.

---

### 3. Jalankan semua layanan

```bash
docker compose up -d
```

---

## Akses UniFi

`https://<IP-SERVER>:8443`

Contoh: `https://192.168.1.38:8443`

---

## First setup (wizard UniFi)

- Buat akun admin aplikasi UniFi
- Atur negara dan timezone
- Lewati restore jika instalasi baru
- Login ke dashboard

---

## Device adoption

URL inform:

`http://<IP-SERVER>:8080/inform`

Contoh: `http://192.168.1.38:8080/inform`

---

## Override inform host (hindari IP internal Docker)

Dengan jaringan **bridge** Docker, UniFi kadang memakai **IP internal container** (misalnya `172.18.0.x`) saat memberi tahu perangkat ke mana harus `inform`. Alamat itu tidak bisa dijangkau dari LAN, sehingga adoption atau status perangkat bisa salah.

Aktifkan **override** agar yang terbaca perangkat adalah **IP atau hostname host Armbian** (sama dengan yang Anda pakai untuk buka `https://…:8443` dan `http://…:8080/inform`), bukan IP container.

1. Login ke UniFi Network: `https://<IP-HOST>:8443`.
2. Buka **Settings** (ikon roda gigi). Lokasi opsi bisa berbeda menurut versi UniFi Network Application; biasanya salah satu dari berikut:
   - **System** → scroll ke **Advanced** → **Override Inform Host**, atau
   - **Devices** → **Config** → cari opsi **Override Inform Host** / inform host / hostname controller untuk adoption (sering dipakai di UI yang mengelompokkan setelan perangkat di sini).
3. Aktifkan **Override Inform Host** (kadang ditulis mirip *inform host override* / *controller hostname* untuk adoption).
4. Isi dengan **IP LAN host** (contoh: `192.168.1.38`) atau **hostname/FQDN** yang bisa di-resolve semua perangkat UniFi di jaringan yang sama.
5. Simpan perubahan. Perangkat yang sudah pernah mengarah ke IP salah mungkin perlu diarahkan lagi ke `http://<IP-HOST>:8080/inform` (SSH `set-inform` pada AP) atau di-adopt ulang dari dashboard.

Catatan: opsi ini mengatur apa yang **diiklankan controller** ke perangkat; port publish di `docker-compose.yml` tetap harus mengikat ke host yang sama (sudah memakai `ports:` ke host).

---

## Restart

```bash
docker compose down
docker compose up -d
```

---

## Troubleshooting

Log container:

```bash
docker logs unifi
docker logs unifi-mongo
```

Cek port:

```bash
ss -tulnp | grep 8443
```

**Adoption gagal / perangkat mengarah ke IP `172.x`:** Pastikan bagian [Override inform host](#override-inform-host-hindari-ip-internal-docker) sudah diisi dengan IP LAN host, lalu set inform ulang pada perangkat jika perlu.

**Autentikasi Mongo gagal / UniFi tidak start setelah ubah password:** User di MongoDB harus di-update juga (atau hapus volume `./mongo` hanya jika Anda siap kehilangan data DB). Setelah Mongo jalan, Anda bisa mengganti password user dengan `db.changeUserPassword` di shell `mongo`, atau buat user baru yang cocok dengan env UniFi.

---

## Notes

- `MONGO_HOST=mongo` mengacu pada nama service di Docker Compose (bukan `127.0.0.1` kecuali Anda pakai `network_mode: host` dan atur jaringan sendiri).
- MongoDB 4.0 dipilih untuk kompatibilitas ARM64 dengan UniFi Network Application.
- Jangan expose port 8443 ke internet tanpa reverse proxy dan TLS yang layak.

---

## Author

Azhar — personal lab setup
