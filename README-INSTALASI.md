# H8Mart Quick-Commerce Ops

Proyek latihan n8n untuk sinkronisasi stok, penerimaan order, asisten merchant, voucher dan refund, serta monitoring. PostgreSQL menyimpan data H8Mart, mock API mensimulasikan layanan stok dan pembayaran, dan Telegram mengirim digest, approval, serta alert. Instalasi baru dimulai dengan data sintetis.

- [Instalasi lokal](#1-instalasi-lokal) untuk mencoba webhook dan workflow dari laptop.
- [Deployment AWS EC2](#2-deployment-aws-ec2) untuk mengakses n8n lewat domain HTTPS dan mencoba Telegram Trigger serta approval di dalam chat.
- [Konfigurasi n8n](#3-konfigurasi-n8n) dan [cara menguji](#4-cara-menguji) berlaku setelah stack di lingkungan pilihan sudah berjalan.

## Persiapan

Clone repo H8Mart atau ekstrak ZIP proyek. Jalankan perintah dari folder yang langsung berisi `compose.yaml`, `.env.example`, `seed/`, `workflows/`, dan `fixtures/`. Untuk menguji semua fitur, siapkan bot Telegram dan API key OpenAI milikmu atau dari instruktur yang dapat mengakses `gpt-6-luna`.

## 1. Instalasi lokal

1. Pasang dan jalankan Docker Desktop di Windows atau macOS. Di Linux gunakan Docker Engine dengan Compose plugin. Pastikan `docker compose version` menampilkan versi yang terpasang. [Unduh Docker](https://docs.docker.com/get-started/get-docker/)
2. Buat `.env` pada instalasi pertama. Jalankan salah satu perintah sesuai terminal.

   ```powershell
   Copy-Item .env.example .env
   ```

   ```bash
   cp .env.example .env
   ```

3. Buka `.env` dan isi password serta token sesuai daftar di bawah. Untuk lokal, biarkan konfigurasi domain dan TLS terkomentar.
4. Jalankan stack dan periksa statusnya. PostgreSQL melakukan seed otomatis saat volume masih kosong.

   ```sh
   docker compose config --quiet
   docker compose up -d
   docker compose ps
   ```

5. Buka [n8n lokal](http://localhost:5690) dan buat akun owner. Buka [Uptime Kuma lokal](http://localhost:3011) untuk membuat akun monitoring. Jika port di `.env` diubah, pakai port baru. Lanjutkan ke [konfigurasi n8n](#3-konfigurasi-n8n).

## 2. Deployment AWS EC2

1. Buat instance EC2 dengan Ubuntu 24.04 LTS x86_64 dan public IPv4. Konfigurasi yang pernah dipakai untuk milestone adalah `t3.small` dengan disk `gp3` 20 GiB. Pilih key pair dan simpan file `.pem` secara privat. Cek Billing dan AWS Budgets sebelum menjalankan instance. [Panduan memulai EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html)
2. Di Security Group, buka TCP `22` hanya dari IP publikmu (`My IP`). Buka TCP `80` dan `443` untuk domain publik. PostgreSQL, mock API, Kuma, dan port internal n8n diakses lewat jaringan privat stack atau SSH tunnel.
3. Buat subdomain DuckDNS atau record `A` di penyedia DNS milikmu, arahkan ke public IPv4 EC2, lalu cek dengan `nslookup DOMAIN_KAMU`. Sambungkan ke server dengan OpenSSH. Ganti `IP_EC2` dan nama file `.pem` pada contoh.

   ```sh
   ssh -i "./h8mart.pem" ubuntu@IP_EC2
   ```

4. Di Ubuntu, pasang Docker Engine dan `docker-compose-plugin` melalui petunjuk **Install using the apt repository**. Gunakan Compose **2.24.4 atau lebih baru** karena override proyek memakai tag `!override`. [Instalasi Docker di Ubuntu](https://docs.docker.com/engine/install/ubuntu/) · [Dokumentasi `!override`](https://docs.docker.com/reference/compose-file/merge/#replace-value)

   ```sh
   sudo docker compose version
   ```

5. Di server, jalankan `git clone URL_REPO h8mart`, lalu `cd h8mart`. Ganti `URL_REPO` dengan URL repo yang kamu terima. Jika proyek dibagikan sebagai ZIP, jalankan `scp -i "./h8mart.pem" "./H8Mart-Milestone3-Siap-Upload.zip" ubuntu@IP_EC2:~/` dari laptop. Setelah masuk kembali lewat SSH, jalankan perintah berikut.

   ```sh
   sudo apt-get update
   sudo apt-get install -y unzip
   unzip ~/H8Mart-Milestone3-Siap-Upload.zip -d ~/h8mart
   cd ~/h8mart/H8Mart-Milestone3-Publik
   ```
6. Pada instalasi pertama, buat `.env` baru di server. Isi password dan token baru sesuai daftar di bawah. Isi `DOMAIN_NAME` dengan hostname tanpa `https://`, `SSL_EMAIL` dengan email pengelola sertifikat, dan `N8N_ENCRYPTION_KEY` dengan hasil `openssl rand -hex 32`. Simpan encryption key bersama cadangan privat karena n8n membutuhkannya untuk membaca kredensial yang sudah tersimpan.

   ```sh
   cp .env.example .env
   chmod 600 .env
   nano .env
   ```

7. Jalankan Compose dengan kedua file. Caddy meminta sertifikat HTTPS untuk domain yang sudah mengarah ke EC2. Ganti `DOMAIN_KAMU` sebelum menjalankan `curl`.

   ```sh
   sudo docker compose -f compose.yaml -f compose.vps.yaml config --quiet
   sudo docker compose -f compose.yaml -f compose.vps.yaml up -d
   sudo docker compose -f compose.yaml -f compose.vps.yaml ps
   curl -fsS https://DOMAIN_KAMU/healthz
   ```

8. Buka `https://DOMAIN_KAMU` di browser dan buat akun owner n8n. Untuk membuka Kuma, jalankan tunnel dari laptop dan biarkan sesi SSH berjalan. Contoh ini memakai port Kuma bawaan di server. Setelah tunnel tersambung, buka [Kuma di VPS melalui tunnel](http://localhost:13011), lalu lanjutkan ke [konfigurasi n8n](#3-konfigurasi-n8n).

   ```sh
   ssh -i "./h8mart.pem" -N -L 127.0.0.1:13011:127.0.0.1:3011 ubuntu@IP_EC2
   ```

### Nilai yang perlu diisi di `.env`

- `PG_SUPERUSER_PASSWORD`, `PG_READER_PASSWORD`, `PG_WRITER_PASSWORD` untuk tiga akun PostgreSQL yang terpisah.
- `MOCK_API_KEY` untuk header `X-Api-Key` pada mock API dan `WEBHOOK_TOKEN` untuk header `X-H8Mart-Token` pada webhook serta MCP.
- `N8N_RUNNERS_AUTH_TOKEN` untuk koneksi n8n ke task runners. Compose memakai nilai yang sama pada kedua service.
- Pertahankan versi image bawaan agar n8n dan task runners tetap kompatibel. Simpan token bot dan API key OpenAI di **Credentials** n8n. Jangan commit `.env`, key SSH, atau export kredensial.

## 3. Konfigurasi n8n

### Buat kredensial

Buat kredensial di **Credentials → Create Credential** pada instance yang akan dipakai. Untuk kedua kredensial PostgreSQL, isi host `postgres`, port `5432`, database `h8mart`, dan SSL **Disable** untuk koneksi internal Compose.

| Nama di workflow | Tipe dan nilai |
| --- | --- |
| H8Mart PG Reader | **Postgres**, user `mcp_reader`, password dari `PG_READER_PASSWORD`. |
| H8Mart PG Writer | **Postgres**, user `ops_writer`, password dari `PG_WRITER_PASSWORD`. |
| H8Mart Mock API | **Header Auth** dengan `X-Api-Key` = `MOCK_API_KEY`. |
| H8Mart Webhook | **Header Auth** dengan `X-H8Mart-Token` = `WEBHOOK_TOKEN`. |
| H8Mart MCP Internal | **Header Auth** dengan `X-H8Mart-Token` = `WEBHOOK_TOKEN`. |
| H8Mart MCP Ops | **Multiple Headers Auth** dengan `X-H8Mart-Token` = `WEBHOOK_TOKEN` dan `X-H8Mart-Merchant` = `ALL`. |
| H8Mart MCP Merchant | **Multiple Headers Auth** dengan `X-H8Mart-Token` = `WEBHOOK_TOKEN`, `X-H8Mart-Merchant` = `{{ $json.merchant_id }}`, dan `X-H8Mart-Date` = `{{ $json.tanggal }}`. Pilih mode **Expression** untuk dua nilai terakhir. |
| H8Mart Telegram | **Telegram API**, masukkan token dari BotFather dan gunakan Base URL bawaan. |
| H8Mart OpenAI | **OpenAI**, masukkan API key yang dapat mengakses `gpt-6-luna`. Di node model, biarkan **Options** kosong. |

### Siapkan Telegram dan Kuma

1. Buat bot melalui `@BotFather` dengan `/newbot`, simpan token di credential Telegram API, lalu kirim `/start` ke bot milikmu. Catat ID chat Ops milikmu. Ikuti [panduan Telegram proyek](guide/telegram-bot.md) untuk memperoleh ID dan menguji pesan.
2. Di Kuma, buat monitor tipe **Push**. Untuk jadwal sinkron tiap 15 menit, gunakan heartbeat interval `1200` detik. Ambil token di URL Push. Workflow menghubungi `http://uptime-kuma:3001/api/push/<token>` dari dalam Docker.
3. Untuk chat asisten di AWS, tambahkan ID chat milikmu ke allowlist dan petakan ke merchant demo. Contoh ini memakai `M003`. Ganti `123456789` dengan ID chat sendiri.

   ```sh
   sudo docker compose -f compose.yaml -f compose.vps.yaml exec -T postgres psql -v ON_ERROR_STOP=1 -U h8mart -d h8mart -c "INSERT INTO ops.telegram_merchants (chat_id, merchant_id, label) VALUES (123456789, 'M003', 'chat demo') ON CONFLICT (chat_id) DO UPDATE SET merchant_id = EXCLUDED.merchant_id, label = EXCLUDED.label;"
   ```

### Impor dan Publish

1. Buat workflow baru di n8n, buka menu **⋯ → Import from File**, lalu pilih JSON dari `workflows/`. Ulangi untuk WF1 sampai WF5. Impor juga WF3b untuk chat merchant dan approval native di instance HTTPS. [Dokumentasi impor n8n](https://docs.n8n.io/workflows/export-import/)
2. Di setiap node yang memakai kredensial, pilih kredensial yang baru kamu buat. Referensi `GANTI-ID-*` di export hanya placeholder. Pada node Telegram WF1, WF4, dan WF5, ganti `ISI_CHAT_ID_OPS` dengan ID chat Ops. Di `Approval Refund Telegram` pada WF4, isi juga **Approver IDs** dengan ID akun Ops yang berwenang. Untuk demo chat pribadi, kedua nilai memakai ID yang sama. Di node `Heartbeat Uptime Kuma` pada WF1, ganti `ISI_TOKEN_PUSH_KUMA` dengan token monitor.
3. Atur **Timezone** ke `Asia/Jakarta`. Publish WF5 dan pilih workflow itu sebagai **Error Workflow** di Settings WF1–WF4 serta WF3b jika diimpor. Publish WF2 untuk membuka endpoint MCP internal, lalu Publish WF1, WF3, dan WF4. Di VPS, Publish WF3b dengan **Telegram Trigger v1.5** memakai bot yang sama dengan node approval. Satu bot memakai satu webhook aktif. [Dokumentasi Publish n8n](https://docs.n8n.io/workflows/publish/)
4. Jalankan WF1 dari trigger **Manual Sinkron** sebelum menguji order. Gunakan **Manual Digest** untuk mencoba laporan. Kedua Schedule Trigger pada export dinonaktifkan. Jika ingin sinkron otomatis tiap 15 menit dan digest setiap pukul 07.00 WIB, aktifkan jadwal yang diperlukan lalu Publish ulang WF1. Monitor Kuma bisa berstatus Down ketika jadwal sinkron tetap mati.

Gunakan nama service untuk koneksi dari node Docker. PostgreSQL ada di `postgres:5432`, mock API di `http://mock-api:8080`, dan endpoint WF2 di `http://n8n:5678/mcp/h8mart-data` dengan transport **HTTP Streamable**. Di VPS, Caddy memblokir MCP dari internet.

## 4. Cara menguji

Jalankan contoh dari root proyek. Ganti base URL dengan `https://DOMAIN_KAMU` untuk menguji VPS dari laptop. Gunakan `WEBHOOK_TOKEN` dari instance yang sedang diuji dan pastikan workflow sudah **Published**.

**Bash**

```bash
base_url='http://localhost:5690'
read -rsp 'Masukkan WEBHOOK_TOKEN: ' token
printf '\n'
curl -sS -i "$base_url/webhook/h8mart-order" \
  -H "X-H8Mart-Token: $token" -H 'Content-Type: application/json' \
  --data-binary @fixtures/event-order-created-ok.json
unset token
```

**PowerShell 7.2+**

```powershell
$baseUrl = 'http://localhost:5690'
$token = Read-Host 'Masukkan WEBHOOK_TOKEN' -MaskInput
Invoke-RestMethod -Method Post -Uri "$baseUrl/webhook/h8mart-order" `
  -Headers @{ 'X-H8Mart-Token' = $token } -ContentType 'application/json' `
  -InFile './fixtures/event-order-created-ok.json'
Remove-Variable token
```

| Skenario | Input | Hasil yang dicari |
| --- | --- | --- |
| Order normal | `/webhook/h8mart-order` dengan `fixtures/event-order-created-ok.json` | HTTP `200`. |
| Stok habis | Endpoint order dengan `fixtures/event-order-created-stok-habis.json` | HTTP `409`. Di PowerShell, `Invoke-RestMethod` menampilkan status ini sebagai error HTTP. |
| Voucher telat | `/webhook/h8mart-event` dengan `fixtures/event-delivery-late-ringan.json` | Voucher Rp10.000 tercatat oleh Payment tiruan. |
| Refund besar | Endpoint event dengan `fixtures/event-order-failed-besar.json` di VPS | `pending_approval`. Akun Ops membuka chat bot dan memilih **Approve** atau **Decline**. |
| Asisten merchant | `/webhook/h8mart-assistant` dengan body `{"merchant_id":"M003","question":"Berapa total pesanan merchant saya kemarin?"}` | Jawaban dari data seed atau eskalasi CS saat data/model tidak memadai. |
| Chat asisten | Kirim pertanyaan yang sama ke bot yang dipakai WF3b pada VPS | Akun yang sudah ada di allowlist menerima balasan. |
| Sinkron dan alert | Jalankan **Manual Sinkron** dan lihat **Executions** serta chat Ops | Pada seed bawaan, 11 merchant berhasil. M009 sengaja gagal untuk menguji retry, dead-letter, dan alert. |

Untuk menguji draft webhook, buka node Webhook, klik **Listen for test event**, lalu kirim request ke **Test URL** `/webhook-test/...`. Jadwal dan **Production URL** memakai versi yang dipublikasikan.

## 5. Operasi dan pemecahan masalah

- **Port bentrok di laptop.** Ubah port host terkait di `.env`, lalu jalankan `docker compose up -d` lagi dan pakai URL dengan port baru.
- **Workflow gagal terhubung ke PostgreSQL.** Dari node n8n gunakan `postgres:5432`. Cek `docker compose ps` dan `docker compose logs --tail=80 postgres`. Password di `.env` dipakai ketika database pertama kali diinisialisasi. Pada volume lama, perubahan `.env` saja tidak merotasi password di database.
- **HTTPS belum berhasil.** Cocokkan record DNS dengan public IPv4 dan periksa Security Group `80/443`. Lihat `sudo docker compose -f compose.yaml -f compose.vps.yaml logs --tail=80 caddy`.
- **MCP gagal atau webhook 403.** Pastikan WF2 sudah Published, endpoint memakai hostname internal, dan credential pengirim serta penerima menggunakan token yang sama.
- **Telegram gagal.** Pastikan pengguna sudah mengirim `/start`, chat ID dan credential cocok, serta bot WF3b terdaftar di instance HTTPS yang dipakai. Untuk pengujian paralel, gunakan bot berbeda.
- **Menghentikan stack sambil menyimpan data.** Jalankan `docker compose stop` di lokal atau `sudo docker compose -f compose.yaml -f compose.vps.yaml stop` di VPS. Gunakan `up -d` untuk menjalankannya kembali. Opsi `down -v` menghapus volume beserta database, workflow, dan kredensialnya.
- **Selesai memakai AWS.** Cadangkan data yang diperlukan sebelum menghapus resource. EC2 dan resource terkait dapat tetap menimbulkan biaya. Periksa volume EBS, public IP yang dialokasikan, dan Billing setelah cleanup. [Panduan cleanup EC2](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/EC2_GetStarted.html#ec2-clean-up)

Dokumentasi tambahan ada di [SETUP.md](SETUP.md), [README solusi](docs/05-readme-solusi.md), [contoh fixture](fixtures/README.md), dan [indeks bukti](evidence/00-index.md).
