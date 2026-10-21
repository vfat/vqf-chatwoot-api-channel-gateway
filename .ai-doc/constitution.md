# Constitution for `vqf-chatwoot-api-channel-gateway`

## Purpose

Konstitusi ini mendefinisikan aturan dan standar dokumentasi serta perencanaan berbasis bukti (*evidence-based documentation & planning*) untuk proyek **vqf-chatwoot-api-channel-gateway**.

Proyek ini merupakan sistem greenfield yang dirancang sebagai gateway / middleware penghubung antara berbagai channel eksternal dengan fitur **API Channel** pada Chatwoot.

Baseline mengadopsi standar umum dari `ai-documentor`.

---

## 1. Workspace Rules

- Ruang kerja dokumentasi dan perancangan berada di `.ai-doc/`.
- File control plane:
  - `.ai-doc/3p.md` (Progress, Plan, Pending tracker)
  - `.ai-doc/constitution.md` (Aturan dan pedoman lokal proyek)
  - `.ai-doc/project-overview.md` (Dokumen perancangan awal sistem greenfield)
- Artefak turunan (`Dokumentasi-Fitur.md`, `Dokumentasi-Komponen-Usecase.md`, `C4-Component-Diagrams.md`, `desain-component-document/`, `desain-database-document/`, `rest-api-doc/`) **hanya dibuat bila diminta secara eksplisit oleh user** (*Only-On-Request Rule*).

---

## 2. Progress Tracking

- File `3p.md` harus selalu dibaca dan diperbarui setelah menyelesaikan langkah bermakna.
- Struktur standar `3p.md`:
  - `Progress`
  - `Plan`
  - `Pending`

---

## 3. Evidence Rules

- Tidak boleh mendokumentasikan klaim atau asumsi tanpa bukti.
- Pada fase Greenfield Planning:
  - Asumsi dan kebutuhan harus ditandai jelas (`Draft`, `Placeholder`, `Asumsi`, atau `Perlu Dikonfirmasi`).
- Pada fase Implementasi & Testing:
  - Setiap klaim harus traceable ke source code, file konfigurasi, atau unit/integration test nyata.

---

## 4. Klasifikasi Komponen & Objek

- **Boundary**: HTTP Handlers, Webhook Ingress (menerima pesan dari channel eksternal atau webhook dari Chatwoot), HTTP Client outbound.
- **Control**: Router / Dispatcher pesan, Session / Conversation State Manager, Channel Adapters, Media / Attachment Transformer.
- **Entity**: Data Transfer Objects (DTO) untuk payload Chatwoot API Channel, skema webhook, konfigurasi channel, dan error definitions. API endpoint diklasifikasikan sebagai `Entity`.
- **Infrastructure**: Runtime, Database/Cache (jika ada untuk mapping source_id & conversation_id), Docker/Deployment environment.

---

## 5. TDD Policy Gate

- Status TDD: *Menunggu Keputusan User* (Perlu Dikonfirmasi).
- Jika diaktifkan: Siklus RED → GREEN → REFACTOR wajib dijalankan dengan bukti test failure yang valid sebelum implementasi.
