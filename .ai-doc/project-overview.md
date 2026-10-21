# Project Overview: VQF Chatwoot API Channel Gateway

> **Dokumen**: Perancangan Awal Sistem Greenfield  
> **Target Sistem**: `vqf-chatwoot-api-channel-gateway`  
> **Workspace**: `/home/ubuntu/workspace/plan/vqf-chatwoot-api-channel-gateway`  
> **Repository**: `https://github.com/vfat/vqf-chatwoot-api-channel-gateway.git`  
> **Standar / Template**: `ai-documentor` (`project-overview-template.md`)  
> **Status**: Draft / Greenfield Discovery  
> **Tanggal**: 2026-10-21  

---

## 1. Problem Statement

Chatwoot menyediakan tipe inbox **API Channel** yang memungkinkan integrasi kustom dengan platform pihak ketiga atau kanal perpesanan eksternal. Namun, untuk menghubungkan kanal eksternal tersebut ke Chatwoot, diperlukan middleware / gateway yang mampu:
1. Menerjemahkan format pesan eksternal menjadi payload percakapan dan pesan Chatwoot (`source_id`, kontak, attachment).
2. Mempertahankan pemetaan identitas sesi/kontak antara kanal eksternal dan percakapan Chatwoot.
3. Menerima webhook event balasan dari agen di Chatwoot dan mendispatch kembali pesan tersebut ke kanal eksternal yang tepat secara andal (*bidirectional bridge*).
4. Menangani otentikasi, validasi signature, rate limiting, dan potensi retry/resilience.

---

## 2. Target Users & Stakeholders

| Role / Pihak | Deskripsi & Kebutuhan |
| :--- | :--- |
| **Customer Support / Agent (Chatwoot)** | Membalas pesan pelanggan dari dashboard terpusat Chatwoot tanpa perlu berganti-ganti aplikasi perpesanan. |
| **End-Users / Customers** | Berinteraksi melalui kanal pilihannya (WhatsApp, Telegram, Webhook/App kustom) secara lancar dan transparan. |
| **AI Assistant / VQF Agent** | Berkolaborasi dengan human-agent melalui otomasi percakapan dan handoff di Chatwoot. |
| **DevOps / Platform Engineer** | Men-deploy dan memantau gateway yang reliabel, hemat sumber daya, dan mudah dikonfigurasi. |

---

## 3. Assumptions

1. **Chatwoot API Channel Inbox**: Chatwoot instance telah dikonfigurasi dengan setidaknya satu inbox bertipe API Channel, memiliki Webhook URL yang mengarah ke gateway ini, dan `api_access_token` yang valid.
2. **Bidirectional Communication**:
   - **Ingress**: Channel Eksternal -> Gateway -> Chatwoot REST API (`/api/v1/accounts/{account_id}/conversations/{conversation_id}/messages` atau penciptaan percakapan baru dengan `source_id`).
   - **Egress**: Agen Chatwoot membalas -> Chatwoot Webhook Event -> Gateway -> Dispatcher ke Channel Eksternal.
3. **Session / Contact Mapping**: Diperlukan mekanisme mapping antara `source_id` di Chatwoot dan ID pengirim/kontak eksternal.

---

## 4. Goals & Objectives

1. **Reliable Bidirectional Gateway**: Menyediakan jembatan dua arah yang stabil antara kanal perpesanan eksternal dan API Channel Chatwoot.
2. **Decoupled Architecture**: Memisahkan logika adapter spesifik kanal eksternal dari logika interaksi inti API Chatwoot.
3. **Resilient Delivery**: Penanganan timeout, retry, dan validasi format payload untuk mencegah pesan hilang atau terduplikasi.
4. **Rich Media Support**: Mendukung pengiriman teks dan lampiran file/gambar/audio secara transparan antar kedua sisi.

---

## 5. Scope

### In Scope (Draft)
- Ingress webhook / listener untuk menerima event pesan dari channel eksternal.
- Chatwoot API Client untuk membuat/mencari kontak, membuat percakapan, dan mengirim pesan inbound.
- Egress webhook listener untuk menerima event `message_created` (outgoing) dari Chatwoot.
- Router/Adapter untuk mengirimkan kembali balasan agen ke channel tujuan.
- Konfigurasi via environment variables / file config.

### Out of Scope (Sementara)
- Modifikasi internal core engine Chatwoot (gateway berjalan mandiri di luar Chatwoot).
- Web UI kompleks (fokus pada headless service / API gateway).

---

## 6. High-Level System Direction

```
+-------------------+        +-----------------------------------+        +-------------------+
|                   |        | vqf-chatwoot-api-channel-gateway   |        |                   |
| External Channel  | ===>   |  [ Ingress Channel Adapter ]      |        |                   |
| (WhatsApp/Custom) |        |              │                    |        |                   |
|                   |        |              ▼                    |        |                   |
|                   |        |  [ Chatwoot API Client (POST) ]   | ===>   | Chatwoot Server   |
|                   |        |                                   |        | (API Channel)     |
|                   |        |  [ Chatwoot Webhook Ingress ]     | <===   |                   |
|                   |        |              │                    |        |                   |
|                   |        |              ▼                    |        |                   |
|                   | <===   |  [ Egress Channel Dispatcher ]    |        |                   |
+-------------------+        +-----------------------------------+        +-------------------+
```

---

## 7. Key Constraints

- Kompatibilitas dengan API Chatwoot v1 (`/api/v1/...`).
- Format `source_id` harus unik dan konsisten untuk setiap kontak eksternal.
- Waktu respons webhook harus cepat (< 2 detik) untuk menghindari timeout dari Chatwoot / provider eksternal (asinkron / queuing jika diperlukan).

---

## 8. Prerequisite

- Instance Chatwoot aktif (lokal atau cloud).
- Konfigurasi Inbox API Channel di Chatwoot beserta credentials (Account ID, API Access Token / User Token).
- Kredensial atau endpoint kanal eksternal yang ingin dihubungkan.

---

## 9. Catatan Diskusi

- Proyek diinisialisasi berdasarkan URL repositori: `https://github.com/vfat/vqf-chatwoot-api-channel-gateway.git`.
- Repositori masih dalam keadaan kosong (greenfield).

---

## 10. Risiko, Asumsi, dan Hal yang Perlu Dikonfirmasi

1. **Tech Stack**: Bahasa pemrograman dan framework yang diinginkan (apakah Golang seperti `vqf-agent`/`whmcs-mcp`, atau Node.js / TypeScript, atau lainnya)?
2. **Channel Target**: Kanal spesifik apa saja yang menjadi prioritas utama gateway ini (misal WhatsApp via Baileys/WPPConnect/Fonnte, Telegram, atau custom webhook generic)?
3. **Penyimpanan State**: Apakah gateway membutuhkan database lokal (SQLite/PostgreSQL) atau Redis untuk menyimpan mapping session/conversation atau cukup stateless memanfaatkan lookup API Chatwoot?
4. **TDD Policy**: Apakah proyek ini akan dikembangkan menggunakan metodologi TDD (Test-Driven Development)?
