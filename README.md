# bnt-relay (personal)

Setup mining Blocknet (BNT) untuk satu pemilik: miner Seine di mesin B, relay stratum di VPS sendiri (mesin A), payout ke alamat BNT sendiri.

Repo ini salinan yang sudah direview dari setup grup teman, dengan perubahan penting:

- `PRODUCTION_ADDRESS` di `setup-complete.sh` dikosongkan. Isi alamat BNT kamu sendiri dulu. Skrip berhenti kalau masih kosong.
- Semua URL download menunjuk ke repo ini sendiri (`n0krayy/bnt-relay`), bukan repo orang lain.
- Binary Seine diambil hanya dari rilis resmi `zpalmtree/seine` (v0.2.24) dan diverifikasi SHA-256 sebelum ekstrak.
- `bakekok.sh` di repo ini defaultnya placeholder (`YOUR_RELAY_IP`, `YOUR_BNT_ADDRESS`). Skrip setup menulis ulang keduanya saat install miner.

## Kenapa pakai relay

Dari sebagian jaringan, koneksi direct ke `bntpool.com:3333` diblokir atau direset di tengah jalan. Relay meneruskan TCP mentah dari port 443 mesin A ke `bntpool.com:3333`, jadi miner di mesin B bicara ke `stratum+tcp://<IP-VPS-KAMU>:443`.

Relay (`relay.py`) hanya meneruskan byte dua arah. Ia tidak membaca atau mengubah isi stratum. Pembatasan siapa yang boleh memakai relay dipegang iptables di mesin A (allowlist IP egress mesin B, sisanya DROP di port 443).

Konsekuensi relay: dev fee Seine lewat relay 2,5% (direct 1%). Fee pool tetap 2,5%, PPLNS 6 jam, minimum payout 0,1 BNT, konfirmasi 60 blok.

## Arsitektur

```
Mesin B (miner, Seine) --stratum tcp 443--> Mesin A (relay.py, VPS kamu) --stratum tcp 3333--> bntpool.com
```

## Yang harus disiapkan

- Mesin A: VPS Linux dengan IP publik IPv4, akses root. IP ini yang dipakai miner sebagai `--relay <IP-A>`.
- Mesin B: Linux 64-bit, akses root, RAM cukup. Tiap thread Seine butuh sekitar 2 GB (HugePages 1024 x 2 MiB untuk konfigurasi bawaan runner).
- Alamat BNT kamu sendiri (atau shortname `@nama` / `$nama` dari blocknet.id).
- IP egress mesin B (hasil `curl -s ifconfig.me` di mesin B) untuk allowlist relay.

## Urutan setup

Detail perintah ada di `COMMANDS.md`. Ringkasnya:

1. Isi `PRODUCTION_ADDRESS` di `setup-complete.sh` dengan alamat BNT kamu.
2. Mesin A: `./setup-complete.sh relay <IP-egress-mesin-B>`
3. Mesin B: `./setup-complete.sh mining --relay <IP-mesin-A> --run`
4. Verifikasi di log miner: `login accepted`, share `accepted` naik, `HugePages_Total` 1024. Saldo dan hashrate bisa dicek di My Stats bntpool pakai alamat kamu.

## File

- `relay.py` - relay TCP :443 ke bntpool.com:3333 (sudah termasuk perbaikan socket blocking setelah connect, supaya sesi tidak putus saat pool diam lebih dari 15 detik).
- `bnt-relay.service` - unit systemd untuk relay.
- `setup-complete.sh` - installer dua peran: `relay` di mesin A, `mining` di mesin B.
- `bakekok.sh` - runner Seine: cek HugePages lalu menjalankan miner dengan POOL_URL dan ADDRESS dari skrip setup.
- `COMMANDS.md` - langkah per mesin.
