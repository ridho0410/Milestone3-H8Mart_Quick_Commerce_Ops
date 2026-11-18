# 00 · Indeks Evidence

Pemeriksaan akhir: 10 November 2026. Data, Stock API, Payment API dan Notification API adalah sintetis. Bukti lokal dan VPS dipisahkan; screenshot tidak diedit untuk mengubah hasil. Mulai dari [README solusi](../docs/05-readme-solusi.md).

Screenshot editor AWS diperbarui 15 November 2026. Bukti hasil run ditautkan pada bagian Executions dan bukti spesifik tiap workflow.

## WF1 — Stok, order dan digest

- [Canvas sinkron stok](wf1/01a-canvas-sinkron-stok.jpg), [canvas order intake](wf1/01b-canvas-order-intake.jpg), [canvas digest Ops](wf1/01c-canvas-digest-ops.jpg), [Executions](wf1/02-executions.jpg), dan [output Merge](wf1/11-merge-output.jpg).
- [Hasil integrasi lokal](wf1/03-local-integration-results.json): dua sinkron, retry M004/M009, order 200/409/422, duplikasi, stok menipis dan stok baru habis.
- [SQL independen](wf1/07-independent-kpi-stock-snapshot.json) dan [chat digest Telegram](wf1/06-telegram-digest.png): 162 order, cancel 15,43%, SLA 82,84% pada hari data terakhir 2026-11-08.
- Screenshot uji mandiri WF1: [order valid](../fixtures/Hasil%20Uji/event-order-created-ok.json.png), [stok habis](../fixtures/Hasil%20Uji/event-order-created-stok-habis.json.png), [stok menipis](../fixtures/Hasil%20Uji/event-order-created-stok-menipis.json.png), [item kosong ditolak](../fixtures/Hasil%20Uji/event-order-created-item-kosong.json.png), dan [jumlah stok 205 baris di DBeaver](../fixtures/Hasil%20Uji/event-order-created-ok.json%20Hasil.png).
- [Sinkron kedua VPS](deploy/09-vps-second-sync.json): 188 baris dari 11 merchant; total tetap 205 pasangan stok. M009 sengaja selalu HTTP 500, dicoba tiga kali lalu dead-letter; ini bukan target 12 merchant sukses.
- [Pemulihan lokal 10 November](wf1/12-local-resume-health.json): container yang berhenti dijalankan kembali tanpa menghapus volume; execution 80 sukses, stok tetap 205. Node Ambil Stok dapat menunjukkan kegagalan M009 saat workflow melanjutkan merchant lain.
- [20 logic tests](wf1/10-logic-test-results.txt), dapat diulang dengan `node --test evidence/wf1/09-logic.test.cjs`.

## WF2 — MCP read-only

- [Canvas](wf2/01-canvas.jpg), [Executions](wf2/02-executions.jpg), [tiga tool lokal](wf2/03-mcp-local-tools.json), dan [INSERT reader ditolak](wf2/04-reader-write-denied.txt).
- [Agent VPS execution 27 memanggil MCP execution 28](wf2/05-vps-agent-mcp-execution.json): nama tool dan parameter nyata dari percakapan merchant.
- [Uji MCP melalui canvas](wf2/06-vps-mcp-canvas-test.json), execution 37, memakai MCP Test URL untuk screenshot canvas hijau. Pemanggilan agent produksi dibuktikan terpisah oleh execution 28.

## WF3 — Asisten AI dan Telegram

- [Canvas](wf3/01-canvas.jpg), [Executions](wf3/02-executions.jpg), [10 kasus dev set dan retest D05](wf3/03-dev-set-results.json), [fallback key dummy](wf3/04-fallback-invalid-key.json), [audit scope/token](wf3/05-tool-scope-token-audit.json), dan [tiket CS](wf3/06-cs-tickets-sql-snapshot.json).
- [Executions Telegram Trigger](wf3/07-telegram-workflow-executions.jpg), [trace chat nyata](wf3/08-telegram-live-chat.json), dan [screenshot balasan 15 order](wf3/11-telegram-live-chat.png): WF3b 26 → WF3 27 → MCP 28 untuk merchant demo M003.
- [Guard chat asing](wf3/09-telegram-foreign-chat-guard.json) memakai clone terkontrol dengan allowlist database asli; node pengiriman diganti pencatatan. Tidak mengirim pesan ke akun asing dan tidak menjalankan agent/LLM. Ini uji guard, bukan percakapan nyata akun kedua.
- Screenshot uji mandiri WF3: [ringkasan run D01–D10](../fixtures/Hasil%20Uji/dev-set-asisten.json.png) dan [uji pertanyaan dengan catatan injeksi](../fixtures/Hasil%20Uji/question-catatan-injeksi.json.png).
- [Uji canvas WF3](wf3/10-vps-canvas-webhook-test.json), execution 38, memakai Test URL.
- Token trace adalah `tokenUsageEstimate`, bukan usage billing aktual. Dokumen 02 menghitung estimasi input dari trace itu, memberi asumsi terpisah untuk output, dan tidak menyebutnya tagihan provider.
- [Spend OpenAI 1–14 November 2026](wf3/13-openai-api-spend-2026-11-01-to-14.png): total US$0,02 pada *Default project* dan *All API sources*; bukti agregat, tidak terisolasi khusus ke H8Mart/WF3.

## WF4 — Voucher, refund dan approval

- [Canvas](wf4/01-canvas.jpg), [Executions](wf4/02-executions.jpg), [fixture lokal](wf4/03-local-fixture-results.json), dan [ledger lokal](wf4/04-ledger-sql-snapshot.json). Bukti lokal memakai implementasi Wait sebelumnya; export terakhir memakai Telegram Send and Wait pada VPS HTTPS.
- [Approval pending](wf4/06-telegram-approval-pending.png) dan [sesudah Decline](wf4/07-telegram-approval-declined.png) adalah HM-VPS-REJECT, execution 25. Agent mengklik Decline di UI Telegram sesuai instruksi pengujian pengguna; tidak ada Payment.
- [Approval berhasil](wf4/08-telegram-approval-approved.png): HM-SP-005 disetujui akun Ops; [bukti Payment VPS](deploy/08-vps-approval-payment.json) mencatat HM-SP-008 gagal sekali lalu retry berhasil, refund Rp80.000 (`RF-00003`), serta retry/duplikasi idempotent. Ledger lokal ada di [04](wf4/04-ledger-sql-snapshot.json).
- HM-SP-004 execution 18 gagal pada konfigurasi Trigger lama. Konfigurasi sudah diperbaiki ke Telegram Trigger v1.5. Retry execution 20 memberi output tanpa identitas responder, sehingga guard menolak dan status tetap pending_approval. Kasus ini tidak diklaim sebagai refund sukses.
- HM-SP-099 execution 22 disetujui, tetapi Payment tiruan sengaja gagal setelah tiga percobaan, lalu failed/dead-letter. [Uji voucher canvas VPS](wf4/09-vps-canvas-voucher-test.json), execution 40, memakai HM-SP-009 Rp20.000.
- Screenshot uji mandiri WF4: [delivery terlambat ringan](../fixtures/Hasil%20Uji/event-delivery-late-ringan.json.png), [delivery terlambat berat](../fixtures/Hasil%20Uji/event-delivery-late-berat.json.png), [order gagal sebelum bayar](../fixtures/Hasil%20Uji/event-order-failed-belum-bayar.json.png), [refund nominal kecil](../fixtures/Hasil%20Uji/event-order-failed-kecil.json.png), [payment gagal](../fixtures/Hasil%20Uji/event-order-failed-payment-down.json.png), dan [retry refund HM-SP-008](../fixtures/Hasil%20Uji/event-order-failed-retry.json.png).

## WF5 — Reliability dan monitoring

- [Canvas WF5 di editor AWS](wf5/01-canvas.jpg) menampilkan Error Trigger, penyusunan alert, Telegram dan jejak Notification. [Executions produksi](wf5/02-executions.jpg) dan [output alert](wf5/06-alert-output.jpg) memuat pemanggilan Error Workflow nyata, termasuk execution 24 dari kegagalan Payment HM-SP-099.
- [Uji lokal](wf5/03-monitoring-local-results.json): sinkron 0 baris, digest kosong, alert ambang durasi; konfigurasi sementara dipulihkan. [Dead-letter lokal](wf5/04-dead-letter-sql-snapshot.json) dan [snapshot VPS](deploy/10-vps-sql-snapshot.json) memuat M009/HM-SP-099.
- [Kuma lokal](wf5/05-kuma-up-down-lokal.jpg), [Kuma VPS](wf5/07-kuma-up-down-vps.jpg), [alert Telegram payment/refund dan approval Ops](wf5/08-telegram-payment-alert.png), [alert sinkron 0 baris](wf5/09-telegram-alert-sinkron-nol.png), [tangkapan kedua alert sinkron 0 baris](wf5/10-telegram-alert-sinkron-nol-highlight.png), serta [perlambatan 12.001 ms > 12.000 ms](wf5/11-telegram-alert-slow-sync.png). Pesan 6.970 ms yang terlihat di riwayat lama berasal dari uji lokal saat ambang diturunkan ke 1 ms; teks notifikasinya tetap menulis ambang produksi 12.000 ms, jadi itu bukan bukti valid perlambatan produksi.
- Interval uji Down 30 detik dipulihkan ke 1.200 detik. Kedua Schedule WF1 dimatikan setelah evidence sesuai panduan; webhook tetap Published. Kuma kemudian Down adalah konsekuensi tidak ada jadwal heartbeat.

## Deployment dan validasi paket

- [Manifest runtime](deploy/00-runtime-manifest.json), [dashboard browser](deploy/01-vps-n8n-published.jpg), [enam service Compose](deploy/02-compose-ps-vps.txt), [sertifikat TLS](deploy/03-tls-certificate.json), dan [Security Group](deploy/04-security-group.json).
- [Smoke eksternal](deploy/05-external-smoke.json) dan [curl dari luar VPS](deploy/06-curl-external.txt). Smoke awal menyimpan riwayat kegagalan setup approval; hasil perbaikan ada pada [08](deploy/08-vps-approval-payment.json).
- [Smoke token penilaian baru](deploy/12-grading-access-smoke.json): token lama 403; token baru 200, AI/MCP dan digest Telegram berhasil. Nilai token hanya di catatan privat dan Credentials n8n.
- [Validasi paket](deploy/11-package-validation.json): file starter terlindungi tetap sama, enam export JSON dengan placeholder, tidak ada pinData atau secret runtime yang diketahui dalam teks publik. Screenshot diperiksa secara visual; ini bukan audit keamanan produksi.
- [Referensi harga](deploy/90-public-price-reference.json) mendukung estimasi TCO dan bukan bukti biaya tagihan aktual.

Panel ringkasan kegagalan n8n mencakup pengujian error yang disengaja dan riwayat setup. Angka itu bukan pengukuran SLA produksi. Backup otomatis/restore belum diuji; API produksi dan HA di luar lingkup milestone.

## Pemeriksaan ulang 15 November 2026

[Evaluasi akhir](../docs/06-evaluasi-akhir-2026-11-15.md) dan [ringkasan hasil terbaru](deploy/13-final-review-2026-11-15.json) memuat status terkini. Enam export disegarkan dari Published AWS, 16 Code memakai Python, dan 19 logic tests menjalankan kode Python terbaru. Dev set D01–D10 ditinjau dari eksekusi yang sudah ada. Pada pemeriksaan ini kedua Schedule WF1 masih aktif. Keterangan nonaktif tanggal 10 November di atas adalah kondisi historis. Kontrak status penolakan WF3 sudah diperbaiki dan diuji pada tindak lanjut di bawah. Paket ZIP lama masih perlu dibuat ulang.

### Perbaikan kontrak WF3

[Bukti perbaikan status](wf3/12-response-contract-fix.json) mencatat tiga uji penolakan aman di Production URL setelah Publish. Execution 265–267 mengembalikan status dijawab tanpa agent, MCP tool, atau tiket CS. Ekspor Downloads sudah diperbarui. Regression test menambah jumlah logic tests menjadi 20, semuanya lulus.
