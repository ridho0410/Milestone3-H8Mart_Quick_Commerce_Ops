# 03 · TCO 12 Bulan dan Deployment · H8Mart Quick-Commerce Ops

**Nama:** Ridho · **Tanggal harga:** 9 November 2026 · **Status:** lokal dan VPS HTTPS teruji; diperiksa kembali 10 November 2026.

## 1. Sumber Harga dan Kurs

| Komponen                          | Nilai                        | Sumber                                                                                                                                                                                    |
| --------------------------------- | ---------------------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| n8n Cloud Pro tahunan             | €50/bulan,10.000 executions  | [n8n pricing](https://n8n.io/pricing/)                                                                                                                                                    |
| EC2 t3.small Linux Shared Jakarta | US$0,0264/jam                | [EC2](https://aws.amazon.com/ec2/pricing/on-demand/) dan [price list regional resmi](https://pricing.us-east-1.amazonaws.com/offers/v1.0/aws/AmazonEC2/current/ap-southeast-3/index.json) |
| gp3 Jakarta                       | US$0,096/GB-bulan            | Price list resmi, SKU SZZSUHFFV7Z5J74G; [EBS](https://aws.amazon.com/ebs/pricing/)                                                                                                        |
| IPv4 publik                       | US$0,005/jam                 | [VPC pricing](https://aws.amazon.com/vpc/pricing/)                                                                                                                                        |
| EUR/IDR                           | 1 EUR=Rp20.208,93            | [ECB IDR](https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/eurofxref-graph-idr.en.html),6 November 2026                                       |
| EUR/USD                           | 1 EUR=US$1,1165              | [ECB USD](https://www.ecb.europa.eu/stats/policy_and_exchange_rates/euro_reference_exchange_rates/html/eurofxref-graph-usd.en.html),6 November 2026                                       |
| Cross USD/IDR                     | 20.208,93/1,1165=Rp18.100,25 | Hasil pembagian referensi ECB, bukan kurs jual bank/kartu                                                                                                                                 |

Asumsi model: 730 jam/bulan, 20 GB storage, backup budget US$1/bulan, transfer budget US$2/bulan. Budget bukan tarif per unit AWS. Pajak, kurs kartu, CPU surplus, kapasitas berlebih dan insiden belum termasuk. Kredit promosi tidak dikurangkan dari TCO 12 bulan agar biaya sesudah kredit habis tetap terlihat.

Dasar lab 16 hari:384×(0,0264+0,005)+20×0,096×16/30=US$13,08 sebelum komponen tambahan. Durasi aktual mengikuti penilaian.

## 2. Perbandingan Opsi

| Opsi                                 | Awal                                                   | Bulanan                                                                                                           | TCO 12 bulan     | Catatan                                                                                                                             |
| ------------------------------------ | ------------------------------------------------------ | ----------------------------------------------------------------------------------------------------------------- | ---------------- | ----------------------------------------------------------------------------------------------------------------------------------- |
| n8n Cloud Pro + backend              | 8 jam ×Rp60.000=Rp480.000                              | €50+US$27,842 backend/budget+1 jam admin=Rp1.574.394                                                              | **Rp19.372.724** | DB/mock/Kuma tetap memerlukan hosting. Cloud tidak langsung menjangkau Compose privat; akses aman tambahan belum dibangun.          |
| AWS VPS + Community                  | 12 jam ×Rp60.000=Rp720.000                             | (730×0,0264+730×0,005+20×0,096+1+2)US$+2 jam admin=Rp623.947                                                      | **Rp8.207.366**  | Sesuai target milestone; satu stack private network, Caddy HTTPS.                                                                   |
| On-prem, server diasumsikan tersedia | 12 jam setup=Rp720.000; tanpa pembelian perangkat baru | 60 W ×24×30/1000×Rp1.700+internet Rp100.000+backup Rp30.000+admin 4 jamRp240.000+alokasi aset Rp100.000=Rp543.440 | **Rp7.241.280**  | Semua angka on-prem adalah asumsi anggaran internal, bukan harga hardware/tarif PLN aktual. Aset dan akses publik belum dibuktikan. |

**Alasan memilih VPS dengan angka TCO:** total 12 bulan VPS Rp8.207.366, yaitu Rp966.086 (13,3%) di atas on-prem asumsi Rp7.241.280. Selisih itu membeli hosting AWS yang selalu tersedia selama instance hidup, jaringan privat Compose, HTTPS publik terkelola Caddy, dan kecocokan dengan rubrik deployment. Biaya VPS Rp11.165.358 (57,6%) lebih rendah daripada n8n Cloud Pro + backend Rp19.372.724. On-prem tetap termurah di tabel hanya jika server, listrik, koneksi, backup, dan akses publik sesuai semua asumsi; ketersediaan aset dan akses publik belum dibuktikan. Karena itu keputusan VPS mengutamakan bukti deployment dan operasional yang benar-benar diuji, dengan biaya lebih tinggi dari on-prem tetapi jauh di bawah Cloud pada asumsi ini.

TCO=awal+12×bulanan. LLM terpisah di dokumen 02. Sync saja 96×30 =2.880 executions/bulan melebihi Starter 2.500, sehingga opsi Cloud memakai Pro. Kapasitas keseluruhan tetap memerlukan pengukuran.

## 3. Keputusan dan Kepatuhan

Dipilih VPS AWS untuk kesesuaian penilaian, komunikasi antarservice tanpa membuka PostgreSQL/mock/Kuma ke internet, dan kontrol runner/jadwal/monitor. On-prem terlihat lebih murah hanya bila asumsi aset dan aksesnya benar. Cloud bukan pengganti langsung jaringan Compose privat starter.

Syarat yang dibaca: [UU 27/2022 Pasal 16 ayat (2) huruf a, b, e, JDIH Komdigi](https://jdih.komdigi.go.id/produk_hukum/view/id/832/t/undangundang%2Bnomor%2B27%2Btahun%2B2022) mengenai pembatasan pengumpulan, kesesuaian tujuan, dan keamanan pemrosesan. Rancangan menggunakan data sintetis, scope merchant, least privilege dan alert minimal. Ini penerapan rancangan, bukan pernyataan seluruh kewajiban hukum produksi terpenuhi.

## 4. Deployment Terverifikasi

| Item           | Nilai/status                                                                                                                                                                                                |
| -------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Instance       | AWS Jakarta, t3.small, Ubuntu 24.04 LTS x86_64, gp3 20 GiB, swap 2 GiB. Instance pengguna dipakai; tidak membuat instance tambahan.                                                                         |
| Service        | Enam service aktif: n8n, task-runners, postgres, mock-api, uptime-kuma, caddy. Versi mengikuti starter.                                                                                                     |
| HTTPS          | DuckDNS dan Caddy; sertifikat dipercaya dan hostname diverifikasi. Health endpoint HTTP 200, curl webhook dari laptop di luar VPS menghasilkan 200/409.                                                     |
| Jaringan       | Port 80/443 publik. SSH dibatasi alamat operator /32. PostgreSQL, mock API, dan Kuma hanya loopback; n8n tanpa host port, dilayani Caddy.                                                                   |
| MCP            | http://n8n:5678/mcp/h8mart-data di jaringan Compose. Endpoint MCP publik diblokir Caddy (403).                                                                                                              |
| Data           | Seed 12 merchant/30 produk/205 stok. Dua sinkron VPS: 188 baris dan 11 merchant sukses per run; jumlah stok tetap 205 karena M009 sengaja gagal.                                                            |
| Metadata n8n   | SQLite pada volume n8n_data; PostgreSQL menyimpan core/ops H8Mart.                                                                                                                                          |
| Credential     | Nilai server tersimpan di Credentials n8n; password/token database dan API khusus server. Token Telegram dan key OpenAI dipindahkan dengan persetujuan pengguna. Nilai rahasia tidak disertakan dalam repo. |
| Workflow       | WF1–WF5 dan WF3b Published; WF5 terpasang sebagai Error Workflow. Chat Telegram dan approval native telah diuji.                                                                                            |
| Jadwal         | Konfigurasi 15 menit dan 07 WIB tersimpan dalam export; kedua Schedule dinonaktifkan setelah bukti diambil. Webhook WF1 tetap aktif sesuai panduan penilaian.                                               |
| Monitoring     | Push monitor VPS diuji Up→Down pada interval 30 detik, lalu dipulihkan menjadi 1.200 detik. Dengan Schedule nonaktif, status Down sesudah batas waktu adalah konsekuensi yang diharapkan.                   |
| Backup/restore | Rencana tersedia di bawah; backup otomatis dan uji restore terisolasi belum dijalankan.                                                                                                                     |

## 5. Runbook

Ikuti SETUP.md bagian 7. Jalankan stack dengan:

```bash
docker compose -f compose.yaml -f compose.vps.yaml up -d
docker compose -f compose.yaml -f compose.vps.yaml ps
```

.env server perlu password PostgreSQL, MOCK_API_KEY, WEBHOOK_TOKEN, runner token, DOMAIN_NAME, SSL_EMAIL, dan N8N_ENCRYPTION_KEY. Pertahankan encryption key yang dipakai saat credential pertama disimpan. .env, catatan privat, session, dan key SSH tetap di luar repo publik.

Untuk setup ulang: buat owner, sembilan credential, monitor Push, dan allowlist merchant. Import JSON; ganti referensi credential serta placeholder chat/token. Publish WF5 lebih dahulu sebelum memasangnya sebagai Error Workflow, lalu WF2, WF1, WF3, WF3b dan WF4. Telegram Trigger harus v1.5 agar callback approval bot yang sama diterima. Satu bot memakai satu webhook aktif; gunakan instance VPS untuk chat.

Untuk backup, simpan dump PostgreSQL, volume n8n_data beserta encryption key, dan data Kuma ke penyimpanan privat terenkripsi. Uji restore ke stack terpisah sebelum menyatakan pemulihan terjamin. Rencana ini belum menjadi proses backup otomatis.

Untuk rollback, hentikan pengirim event dan Unpublish workflow yang bermasalah; pertahankan volume dan ledger. Periksa status Payment sebelum replay dengan idempotency key yang sama. Jangan menghapus data sebagai langkah pemulihan rutin.

Akun lab, kredit/masa aktif dan Budget diperiksa pengguna. Budget adalah peringatan biaya. VPS tetap hidup sampai instruktur menyatakan penilaian selesai. Setelah itu pengguna mengganti token penilaian, menghapus key OpenAI dari VPS, terminate resource, dan memeriksa Billing.

## 6. Bukti dan Pengumpulan

- evidence/deploy/02-compose-ps-vps.txt: enam service.
- evidence/deploy/03-tls-certificate.json: TLS dan hostname valid.
- evidence/deploy/04-security-group.json: akses jaringan.
- evidence/deploy/05-external-smoke.json dan 06-curl-external.txt: pengujian dari luar VPS.
- evidence/deploy/08-vps-approval-payment.json: callback Ops, ledger, retry, duplikasi dan Payment gagal.
- evidence/deploy/09-vps-second-sync.json: sinkron kedua dan stok tetap 205.
- evidence/deploy/12-grading-access-smoke.json: token khusus penilaian baru, AI/MCP/digest berhasil; token lama ditolak.
- evidence/00-index.md: peta bukti, lingkup lokal/VPS dan penjelasan riwayat error.
