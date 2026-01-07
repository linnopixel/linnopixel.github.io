# BIND9 Debian 11

Panduan **ringkas, rapi, dan langsung eksekusi** untuk instalasi serta konfigurasi **DNS Server BIND9** di **Debian 11 (Bullseye)**. Semua perintah **dipisah per-blok** agar aman saat copy–paste.

---

## 📦 Repository Debian 11 (Official)

Gunakan repository **resmi Debian 11** berikut.

Edit source list:

```bash
nano /etc/apt/sources.list
```

Isi repository:

```text
deb http://deb.debian.org/debian bullseye main contrib non-free
deb http://deb.debian.org/debian bullseye-updates main contrib non-free
deb http://security.debian.org/debian-security bullseye-security main contrib non-free
```

Update repository:

```bash
apt update
```

---

## 🧩 Informasi Dasar

| Item   | Nilai          |
| ------ | -------------- |
| OS     | Debian 11      |
| Domain | linnopixel.com |
| IP DNS | 192.168.1.10   |

---

## 1️⃣ Instalasi

Update sistem:

```bash
apt update
```

Install BIND9:

```bash
apt install bind9 bind9utils bind9-dnsutils -y
```

Aktifkan service:

```bash
systemctl enable bind9
```

Jalankan service:

```bash
systemctl start bind9
```

Cek status:

```bash
systemctl status bind9
```

---

## 2️⃣ Konfigurasi Utama

Buka file konfigurasi:

```bash
nano /etc/bind/named.conf.options
```

Isi konfigurasi:

```conf
options {
    directory "/var/cache/bind";

    recursion yes;
    allow-query { any; };

    forwarders {
        8.8.8.8;
        1.1.1.1;
    };

    dnssec-validation auto;
    listen-on { any; };
    listen-on-v6 { none; };
};
```

---

## 3️⃣ Forward Zone (Domain → IP)

Edit file zone lokal:

```bash
nano /etc/bind/named.conf.local
```

Tambahkan konfigurasi zone:

```conf
zone "linnopixel.com" {
    type master;
    file "/var/cache/bind/db.linnopixel.com";
};
```

Buat file zone:

```bash
nano /var/cache/bind/db.linnopixel.com
```

Isi file zone:

```dns
$TTL 604800
@   IN SOA ns.linnopixel.com. admin.linnopixel.com. (
        2026010701
        604800
        86400
        2419200
        604800 )
;
@   IN NS  ns.linnopixel.com.
ns  IN A   192.168.1.10
www IN A   192.168.1.10
```

---

## 4️⃣ Reverse Zone (IP → Domain)

Tambahkan reverse zone:

```conf
zone "1.168.192.in-addr.arpa" {
    type master;
    file "/var/cache/bind/db.192";
};
```

Buat file reverse:

```bash
nano /var/cache/bind/db.192
```

Isi file reverse:

```dns
$TTL 604800
@   IN SOA ns.linnopixel.com. admin.linnopixel.com. (
        2026010701
        604800
        86400
        2419200
        604800 )
;
@   IN NS  ns.linnopixel.com.
10  IN PTR ns.linnopixel.com.
```

---

## 5️⃣ Validasi & Restart

Cek konfigurasi utama:

```bash
named-checkconf
```

Cek zone:

```bash
named-checkzone linnopixel.com /var/cache/bind/db.linnopixel.com
```

Restart BIND9:

```bash
systemctl restart bind9
```

---

## 6️⃣ Testing

Test dengan dig:

```bash
dig linnopixel.com
```

Test dengan nslookup:

```bash
nslookup www.linnopixel.com 192.168.1.10
```

---

## 7️⃣ Firewall

Izinkan DNS:

```bash
ufw allow 53
```

Reload firewall:

```bash
ufw reload
```

---

## 8️⃣ Catatan Penting

* Setiap perubahan zone → **wajib naikkan serial**
* Pastikan port **53 TCP & UDP terbuka**
* Jika error, cek log:

```bash
journalctl -u bind9 -xe
```

---

✅ **SELESAI — DNS Server linnopixel.com siap digunakan**

Dokumen ini siap dipakai sebagai **README GitHub versi rapi & profesional**.
