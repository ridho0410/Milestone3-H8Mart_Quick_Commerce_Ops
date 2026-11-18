# Evaluasi akhir H8Mart Milestone 3

Diperiksa 15 November 2026, zona Asia/Jakarta. Penilaian ini memakai skenario laboratorium dan API sintetis.

## Kesimpulan

Implementasi inti berfungsi dan bukti terbaru menunjukkan alur utama berjalan. Kontrak respons penolakan WF3 sudah diperbaiki dan diuji melalui tiga request production. Pengumpulan masih memerlukan penonaktifan dua Schedule WF1 setelah evidence dan pembuatan ulang paket pengumpulan. Nilai instruktur tidak diperkirakan karena hidden test belum dijalankan.

## Cakupan dan perubahan

- Folder asli OneDrive dibaca sebagai acuan. File starter pada salinan Downloads dibandingkan dengan sumber dan tidak ditemukan perbedaan.
- README, kontrak data, panduan WF1–WF5, tugas, pengumpulan, lima dokumen solusi, ekspor, dan indeks evidence ditinjau.
- Enam ekspor workflow di Downloads disegarkan dari versi Published AWS. Semua 16 node Code memakai pythonNative. Referensi credential, chat ID Ops, dan token push diganti placeholder.
- Harness evidence/wf1/09-logic.test.cjs diperbarui agar menjalankan kode Python ekspor. Seluruh 19 pengujian lulus. Node.js dan Python perlu tersedia. Variabel lingkungan PYTHON dapat menunjuk executable Python yang dipakai.
- Tidak ada perubahan konfigurasi AWS atau transaksi bisnis baru pada review ini.

## Temuan sebelum dikirim

1. **Kontrak WF3 diselesaikan pada 15 November.** Node Balas Penolakan kini mengirim HTTP 200 dengan status dijawab dan pesan penolakan aman. Status eskalasi_cs digunakan oleh cabang tiket CS. Perubahan sudah Published di AWS dan tersimpan dalam ekspor Downloads. Tiga request production (D09 lintas merchant, pertanyaan di luar topik, dan instruksi hapus database) melewati cabang penolakan tanpa agent, MCP tool, atau tiket CS. Bukti ada di evidence/wf3/12-response-contract-fix.json. Pengujian lokal kini berjumlah 20 dan semuanya lulus.
2. **Dua Schedule WF1 aktif.** Jadwal Sinkron 15 Menit dan Jadwal Digest 07 WIB aktif pada snapshot. Panduan guide/submission.md meminta keduanya di-Deactivate lalu Publish setelah bukti terkumpul. Jadwal sudah terbukti berjalan. Ada 61 dead-letter M009 dengan attempts 3 pada snapshot, sesuai merchant yang sengaja selalu gagal. Mematikan jadwal sesuai panduan akan menghentikan heartbeat terjadwal dan monitor proses dapat Down.
3. **Paket ZIP lama tertinggal.** H8Mart-Milestone3-Siap-Upload.zip yang dibuat sebelumnya belum dibangun ulang dari perubahan Python dan harness terbaru. Gunakan isi set_1 yang sudah diperiksa, kemudian buat ulang paket bersih setelah temuan runtime diselesaikan. Hindari mengunggah folder induk karena berisi .pem, .env, session dan catatan kredensial privat.
4. **Dokumentasi operasional perlu pembaruan.** Indeks dan dokumen solusi utama bertanggal 10 November serta menyebut jadwal VPS nonaktif. Kondisi 15 November menunjukkan jadwal aktif. Lima dokumen inti tersedia, tetapi klaim status harus mengikuti kondisi saat serah terima.

## Hasil verifikasi langsung

- HTTPS/healthz menghasilkan 200 dengan validasi sertifikat oleh client. Endpoint MCP dan mcp-test publik menghasilkan 403. Ketiga webhook tanpa header ditolak 403.
- Enam workflow Published dan activeVersionId cocok dengan versionId. WF1–WF4 serta WF3b merujuk Error Workflow WF5. Timezone tersimpan Asia/Jakarta.
- Caddy, n8n, task-runners, mock-api, PostgreSQL dan Uptime Kuma berjalan. PostgreSQL dan Kuma healthy. Port database 5440, mock API 8080 dan Kuma 3011 terikat ke loopback VPS.
- core.merchant_stock berisi 205 baris. Tiga sinkron terakhir mencatat 12 merchant total, 11 berhasil, 188 baris disimpan, durasi sekitar 8,6 detik.
- Hari data terakhir 2026-11-08. M003 memiliki 15 order, 9 batal/gagal dan 6 terkirim. M002 memiliki 13 order dengan 1 batal/gagal. M012 memiliki 15 order terkirim.
- Ledger memuat voucher issued Rp10.000/Rp20.000, refund refunded, keputusan rejected, dan HM-SP-099 failed. HM-SP-004 masih pending_approval sebagai riwayat uji lama. Tidak diklaim sebagai refund sukses.
- Empat puluh eksekusi terbaru pada snapshot berstatus success. Status success tetap perlu dicocokkan dengan keluaran bisnis, sebagaimana pemeriksaan D01–D10 berikut.

## Dev set WF3 dari eksekusi yang sudah ada

Hasil berikut dibaca dari execution detail AWS. Review ini tidak mengirim ulang sepuluh request. Status lulus di sini mencakup isi jawaban agent, cabang respons, dan tool yang tersedia di trace. Temuan kontrak status D09 sudah ditutup melalui uji production lanjutan pada execution 265.

- D01 — execution 246, sesuai pemeriksaan fixture.
- D02 — execution 249, sesuai pemeriksaan fixture.
- D03 — execution 251, sesuai pemeriksaan fixture.
- D04 — execution 253, sesuai pemeriksaan fixture.
- D05 — execution 255, sesuai pemeriksaan fixture.
- D06 — execution 257, sesuai pemeriksaan fixture.
- D07 — execution 258, sesuai pemeriksaan fixture.
- D08 — execution 259, sesuai pemeriksaan fixture.
- D09 — execution 260, sesuai pemeriksaan fixture.
- D10 — execution 261, sesuai pemeriksaan fixture.

D05 memanggil tool Status_Stok untuk scope merchant. D06–D08 melewati Buat Tiket CS dan Balas Eskalasi CS. D09 berhenti sebelum agent dan MCP Client Tool. D10 tidak menjanjikan kompensasi dan confidence di bawah 1. D02 untuk M001 menghasilkan angka berbeda secara benar karena fixture D02 ditujukan ke M003.

## Kesesuaian tujuh bagian rubrik

- Business Requirement & Solution Design tersedia pada dokumen 01–02, meliputi sumber masalah, as-is/to-be, titik manusia/AI, kebijakan, dan ROI berasumsi.
- n8n Workflow Build memiliki trigger, Merge, Switch/If, Python Code, upsert, dan pengaturan zona waktu. Kondisi Schedule perlu disesuaikan dengan fase pengumpulan.
- System Integration & MCP memiliki tool read-only berparameter, role reader/writer, API tiruan dan Telegram. Bukti penolakan INSERT reader berasal dari pengujian sebelumnya.
- AI on Automation memiliki Structured Output Parser, scope merchant, confidence threshold, fallback ke tiket, dan dev set. Status penolakan sudah diselaraskan dan diuji setelah review awal.
- Reliability & Monitoring mencakup retry, dead-letter, Error Workflow, alert, heartbeat, silent failure dan durasi. Uji silent failure dan Kuma Up/Down menggunakan bukti terdahulu, sedangkan container dan data sinkron diperiksa kembali.
- Deployment, Cost & Governance memiliki VPS HTTPS, tiga opsi TCO dengan asumsi dan sumber bertanggal, pembatasan port, credential placeholder dan checklist. Harga lama tidak diverifikasi ulang pada review ini. Token billing aktual dan restore belum dibuktikan.
- Workflow Quality & Documentation memiliki penamaan deskriptif, Sticky Note, lima dokumen dan evidence bernomor. Ekspor Python kini terbaru, paket ZIP dan keterangan jadwal masih perlu finalisasi.

## Pemeriksaan yang belum selesai

- Uji POST baru untuk dev set, voucher dan skenario CS ditolak review persetujuan otomatis karena side effect pada VPS. Uji baru memerlukan izin khusus. Bukti eksekusi yang sudah ada digunakan untuk melanjutkan evaluasi.
- Tidak dilakukan simulasi gangguan baru, klik approval Telegram, restore backup, load test, atau audit keamanan menyeluruh.
- Security Group terkini dan Billing AWS tidak dibaca dari API AWS. Binding port container diperiksa langsung, sedangkan bukti Security Group menggunakan evidence terdahulu.
- Pemindaian ekspor dilakukan terhadap nilai privat yang diketahui dan pola key. Semua screenshot, riwayat Git dan isi paket ZIP belum diaudit ulang.

## Langkah serah terima

Kontrak respons WF3 sudah diselesaikan, Published, dan ekspornya diperbarui. Selesaikan fase jadwal, ekspor perubahan jadwal, lalu bangun ulang paket. Jalankan smoke akhir yang disetujui. Pengguna mengunggah isi set_1 ke repository publik dan mengirim URL serta token penilaian lewat DM privat sesuai pilihannya. VPS tetap hidup sampai instruktur menyatakan penilaian selesai.

## Tindak lanjut perbaikan WF3

Perubahan lanjutan hanya mengganti nilai status pada node Balas Penolakan dari ditolak menjadi dijawab. Koneksi dan pengaturan WF3 dipertahankan. Bukti tiga uji production, execution ID 265–267, dan hasil 20 tes lokal ada pada evidence/wf3/12-response-contract-fix.json. Pernyataan mengenai pemeriksaan read-only dan 19 tes pada review awal merupakan catatan sebelum perbaikan lanjutan ini.
