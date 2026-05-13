# UniFi Network on Armbian (ARM64) via Docker

Deploy UniFi Network Application di Armbian ARM64 menggunakan Docker + MongoDB.

---

## 🚀 Features

- UniFi Network Application
- MongoDB 4.0 compatible ARM64
- Persistent storage
- Auto restart
- Host networking (recommended for adoption)
- Suitable for ARM64 (Armbian / SBC)

---

## 📦 Requirements

- Armbian / Debian-based OS (ARM64)
- Docker
- Docker Compose
- Minimal 2GB RAM (recommended 4GB)

Install Docker:

curl -fsSL https://get.docker.com | sh

Install Docker Compose:

apt install docker-compose -y

---

## 📁 Folder Structure

unifi/
├── docker-compose.yml
├── config/
└── mongo/

---

## ⚙️ Installation

### 1. Create folder

mkdir unifi
cd unifi

---

### 2. Create docker-compose.yml

services:
  mongo:
    image: mongo:4.0
    container_name: unifi-mongo
    restart: unless-stopped
    volumes:
      - ./mongo:/data/db

  unifi:
    image: lscr.io/linuxserver/unifi-network-application:latest
    container_name: unifi
    restart: unless-stopped

    depends_on:
      - mongo

    network_mode: host

    environment:
      - PUID=1000
      - PGID=1000
      - TZ=Asia/Jakarta

      - MONGO_USER=unifi
      - MONGO_PASS=unifi123
      - MONGO_HOST=127.0.0.1
      - MONGO_PORT=27017
      - MONGO_DBNAME=unifi

    volumes:
      - ./config:/config
      - ./mongo:/data/db

---

### 3. Start container

docker compose up -d

---

## 🌐 Access UniFi

https://<IP-SERVER>:8443

Contoh:
https://192.168.1.38:8443

---

## 👤 First Setup

- Create admin user
- Set country & timezone
- Skip restore (jika fresh install)
- Login dashboard

---

## 📡 Device Adoption

Inform URL:

http://<IP-SERVER>:8080/inform

Contoh:
http://192.168.1.38:8080/inform

---

## 🔁 Restart

docker compose down
docker compose up -d

---

## 🧠 Troubleshooting

Check logs:

docker logs unifi
docker logs unifi-mongo

Check port:

ss -tulnp | grep 8443

---

## ⚠️ Notes

- Gunakan `network_mode: host` untuk adoption stabil
- MongoDB 4.0 dipakai untuk kompatibilitas ARM64
- Jangan expose port 8443 ke internet tanpa proxy/SSL

---

## 📌 Author

Azhar - personal lab setup
