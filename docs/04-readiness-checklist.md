# 04 · Governance dan Integration Readiness

**Nama:** Ridho · **Diperbarui:** 10 November 2026 · **Lingkup:** laboratorium sintetis, lokal dan VPS HTTPS.

## 1. Credential dan Akses

| Pemeriksaan | Status | Bukti/keterangan |
| --- | --- | --- |
| Credential database/API/Telegram/OpenAI | Siap lokal dan VPS | Nilai disimpan di Credentials n8n. Export publik hanya memakai referensi placeholder. |
| Pemisahan rahasia server | Siap | Password database, token API/webhook dan encryption key server tersendiri. Catatan privat disimpan di luar root repo. |
| .env di-ignore; session/key SSH di luar root repo | Siap | Pemeriksaan Git dan pemindaian rahasia sebelum paket akhir. |
| mcp_reader read-only | Teruji lokal | evidence/wf2/04-reader-write-denied.txt. |
| ops_writer | Terpasang | WF1/WF3/WF4/WF5 memakai writer. Status refund dicatat pada ops.refund_actions sesuai grant starter. |
| Header Auth webhook | Teruji | Tanpa header ditolak 403; hasil lokal dan evidence/deploy/05-external-smoke.json. |
| Token khusus penilaian | Teruji 10 November | Token baru disimpan privat; token lama 403, token baru 200 dan AI/MCP/digest berhasil. evidence/deploy/12-grading-access-smoke.json. |
| Scope merchant MCP | Teruji | Scope SQL berasal dari header workflow; model tidak memasok merchant_id. Audit WF3 dan pertanyaan lintas merchant diblokir. |
| Allowlist Telegram | Teruji di VPS | Chat Ops dipetakan ke merchant demo M003. Eksekusi Telegram Trigger berhasil dan jawaban sesuai SQL. Uji chat asing dicatat terpisah sesuai lingkupnya. |
| Approval Telegram | Teruji di VPS | Send and Wait, approver ID dibatasi ke Ops. Output tanpa identitas Ops ditolak; tidak memanggil Payment. |
| Port dan TLS | Teruji | evidence/deploy/03-tls-certificate.json dan 04-security-group.json. SSH /32; hanya HTTP/HTTPS publik; MCP publik 403. |
| Token/chat ID dalam export | Disanitasi | ISI_TOKEN_PUSH_KUMA, ISI_CHAT_ID_OPS dan GANTI-ID-* dipetakan pada runtime. |

## 2. Kesiapan Integrasi

| Sistem | Integrasi dan batas | Bila gagal | Penanggung jawab |
| --- | --- | --- | --- |
| Stock | Header Auth; mock M004 sementara gagal, M007 lambat, M009 selalu gagal. SLA/rate limit produksi belum tersedia. | Retry 3, dead-letter, alert dan lanjut merchant lain. Sinkron 0 baris gagal dan tidak mengirim heartbeat. | Ops/engineering; kontak provider produksi belum tersedia. |
| Payment | API tiruan, Idempotency-Key dan ledger unik. State gateway mock berada di memori. | Retry 3, status failed dan dead-letter. Rekonsiliasi sebelum replay. | Finance/engineering. |
| Notification | POST /notify sebagai jejak sintetis. | Error/trace dapat ditelusuri; bukan pesan pelanggan nyata. | Engineering. |
| Telegram | Credential native, Trigger v1.5, bot yang sama untuk chat dan approval. Tidak ada jaminan waktu nyata. | Error terlihat; keputusan tanpa responder Ops ditolak. Pesan terkirim belum berarti refund disetujui. | Ops/pemilik bot. |
| PostgreSQL | Reader/writer terpisah; satu host lab. | Error atau eskalasi AI; HA dan pemulihan belum dibuktikan. | Engineering. |
| OpenAI | Key instruktur, model gpt-6-luna; Options model kosong. Kuota dibagi. | Error output menuju tiket CS dan HTTP 200 eskalasi_cs. | Pemilik key/instruktur. |

## 3. Checklist Sebelum Pengumpulan

| Pemeriksaan | Status | Bukti/keterangan |
| --- | --- | --- |
| WF1–WF5 dan WF3b Published | Siap di VPS | Manifest deployment dan daftar Executions; WF5 dipasang sebagai Error Workflow. |
| Sinkron VPS minimal sekali | Teruji dua kali | 188 baris/11 merchant sukses per run; total stok 205. evidence/deploy/09-vps-second-sync.json. |
| Retry dan dead-letter | Teruji | M004/M009 dan HM-SP-099; evidence WF1/WF4/WF5 serta snapshot VPS. |
| Voucher/refund/duplikasi | Teruji | Voucher Rp10.000/Rp20.000, refund captured sesuai total, duplikasi satu tindakan; ledger dan gateway tiruan. |
| Approval/Decline | Teruji | HM-SP-005 disetujui dan refunded; HM-VPS-REJECT ditolak tanpa Payment; HM-SP-099 failed sesudah approval dan retry. |
| Approval tanpa identitas | Teruji gagal tertutup | HM-SP-004 execution 20 ditolak guardrail dan tetap pending_approval untuk pemeriksaan Ops; tidak diklaim sebagai refund sukses. |
| Chat merchant | Teruji di VPS | Telegram → WF3 → MCP → jawaban M003 sebanyak 15 order pada hari data terakhir. |
| Dua kegagalan diam-diam dan durasi | Teruji lokal | evidence/wf5/03-monitoring-local-results.json. Konfigurasi sementara dipulihkan sesudah pengujian. |
| Heartbeat Up/Down | Teruji lokal dan VPS | Screenshot Kuma; interval uji 30 detik dikembalikan menjadi 1.200 detik. |
| AI dev set/injeksi/fallback | Teruji lokal | Sepuluh response checks sesuai setelah retest D05. Fallback key dummy pada clone terpisah. Sampel lab bukan jaminan semua pertanyaan. |
| Token aktual/billing | Belum terukur penuh | Hanya tokenUsageEstimate tersedia; biaya ROI memakai asumsi eksplisit. |
| Log dan data pribadi | Sesuai lingkup sintetis | Payload dead-letter minimal dan export disanitasi. Kebijakan retensi/masking untuk data nyata perlu audit tersendiri. |
| Backup otomatis/restore | Belum | Rencana dokumen 03; restore terisolasi belum dijalankan. |
| Rollback | Rencana siap | Pertahankan volume/ledger; hentikan event; rekonsiliasi status Payment sebelum replay. |
| HTTPS/curl eksternal | Teruji | evidence/deploy/03, 05 dan 06. |
| Dua Schedule WF1 | Nonaktif sesuai panduan | Definisi 15 menit/07 WIB tetap tersimpan; webhook WF1 tetap Published. |
| Upload GitHub dan DM instruktur | Dilakukan pengguna | Paket akhir disiapkan untuk diunggah sendiri; token penilaian diberikan lewat DM privat. |

## 4. Keputusan Readiness

Siap untuk demonstrasi dan smoke test milestone pada data sintetis. Stock, Payment dan Notification tetap API tiruan. Gateway produksi, SLA provider, idempotensi gateway persisten, HA, backup otomatis dan restore belum menjadi kemampuan yang dibuktikan. VPS dijaga aktif sampai penilaian selesai; penghentian dan pembersihan key dilakukan setelah instruktur memberi tahu.
