# SRS - LifeOS (IEEE 830)

| | |
|---|---|
| Versi | 1.0.0 (2026-09-18) |
| Acuan | BRD, PRD, FSD v1.0.0 |

## 1. Pendahuluan

### 1.1 Tujuan

SRS ini menetapkan kebutuhan perangkat lunak sistem LifeOS (API + web + QA + data + ops) sebagai acuan pembangunan, pengujian, dan serah terima. Audiens: developer, QA, operator (pemilik merangkap semua peran).

### 1.2 Ruang Lingkup

Produk bernama **LifeOS**: REST API personal operating system single-user (Go/Gin + SQLite + JWT, tanpa Docker) beserta web 13 halaman, suite QA, ETL analitik mingguan, dan paket operasional. Manfaat: satu dashboard harian untuk tugas, goals, habit, keuangan, kesehatan, dan refleksi. Konsisten dengan BRD/FSD.

### 1.3 Definisi & Singkatan

| Istilah | Arti |
|---|---|
| Single-user | Hanya 1 akun pemilik; register kedua ditolak 403 |
| Full-replace | PUT mengosongkan field yang tak dikirim (by design) |
| Streak | Rangkaian hari berurutan habit tercatat done |
| Heatmap | Grid 30 hari status habit |
| Cashflow | Selisih income - expense per bulan |
| Ranking terbobot | SUM(weight x score) per opsi keputusan |
| WIB | Asia/Jakarta (UTC+7) |
| RBAC | Tidak berlaku (single-user) |

### 1.4 Referensi

1. BRD v1.0.0, PRD v1.0.0, FSD v1.0.0 (repo ini).
2. `api/swagger.yaml`, `migrations/00001-00015`, ADR-001-sqlite-no-orm - repo `lifeos`.
3. `test-plan.md`, `data/testcases.yaml` (61 TC), `data/bugs.yaml` (4 fixed) - repo `lifeos-qa`.
4. `RUNBOOK.md`, `SLA.md`, `TROUBLESHOOTING.md`, `tickets.yaml` (10 tiket) - repo `lifeos-ops`.

### 1.5 Overview

Bagian 2 deskripsi umum; bagian 3 kebutuhan rinci (antarmuka, fungsional FR, non-fungsional NFR, batasan CON); bagian 4 glosarium; bagian 5 referensi; bagian 6 matriks traceability.

## 2. Deskripsi Umum

### 2.1 Perspektif Produk

Sistem mandiri single-user: API (`:8080`), SQLite file lokal, web (`:5174`), pipeline ETL offline mingguan, tooling QA/ops. Antarmuka: REST JSON; komunikasi HTTP; operasi via runbook (migrate + seed + run); instalasi tanpa Docker.

### 2.2 Fungsi Produk (ringkas)

Auth JWT single-user; dashboard agregat; tasks/projects; goals/milestones; habits/logs; transactions/budgets/subscriptions; health/workouts/learning/reading; reviews 3 period; trips/itinerary/packing; decisions/options/marks; savings/assets/wishlist/documents/contacts; reminders; kalender; ETL; runbook/SLA/tiket.

### 2.3 Karakteristik Pengguna

Satu pengguna teknis (developer) yang memakai sistemnya sendiri. Tak ada kebutuhan aksesibilitas khusus di v1.0.

### 2.4 Lingkungan Operasi

| Aspek | Nilai |
|---|---|
| Runtime API | Go 1.27/Gin, `modernc.org/sqlite` pure-Go (tanpa CGO) |
| Database | SQLite file lokal `lifeos.db` (tidak di-commit) |
| Web | Vite + React 19 + TS (dev `:5174`) |
| Tooling QA | Node 22 (Newman, Playwright) |
| Tooling data | Python 3.13 + pandas/duckdb/matplotlib, timer systemd Senin 06:00 |
| Zona waktu | Asia/Jakarta (UTC+7) di semua komponen |
| Orkestrasi | Tanpa Docker by design; run via runbook (goose + seed + run) |

### 2.5 Batasan

- CON-TECH-01: SQLite file lokal; tanpa Docker; Go; timezone Asia/Jakarta.
- CON-TECH-02: PUT full-replace di semua modul ber-PUT.
- CON-SEC-01: JWT_SECRET >= 32 char di produksi; `.env` dan `lifeos.db*` tidak di-commit.
- CON-STD-01: mengikuti FSD bagian 2-5.

### 2.6 Asumsi & Dependensi

- Single-user by design (multi-user out of scope).
- Seed tanggal tetap Sep 2026; metrik dinamis dihitung dari hari berjalan.
- Bergantung pada: `modernc.org/sqlite`, goose, Go toolchain (CI), Node 22 (CI web/QA), Python 3.13 + pandas/duckdb/matplotlib (CI data).

## 3. Kebutuhan Khusus

### 3.1 Kebutuhan Antarmuka Eksternal

| ID | Kebutuhan |
|---|---|
| FR-INT-01 | API wajib REST JSON dengan kode status & body error terdokumentasi (FSD bagian 2). |
| FR-INT-02 | Web wajib membaca base URL dari `VITE_API_URL` dan mengirim `Authorization: Bearer`; 401 wajib auto-logout. |
| FR-INT-03 | ETL wajib read-only terhadap DB operasional (`mode=ro`) dan menulis ke DuckDB/parquet terpisah. |

### 3.2 Kebutuhan Fungsional

| ID | Kebutuhan (M = wajib) | Prioritas | Sumber |
|---|---|---|---|
| FR-AUTH-01 | Sistem wajib menolak register kedua dengan 403 `single_user_only`. | M | US-01 |
| FR-AUTH-02 | Sistem wajib menerbitkan JWT 24 jam saat login valid; token tak valid -> 401. | M | US-01 |
| FR-DASH-01 | Sistem wajib menyajikan agregat dashboard dalam 1 panggilan. | M | US-02 |
| FR-TASK-01 | Sistem wajib CRUD tugas dengan enum status/prioritas dan auto-stamp completed. | M | US-03 |
| FR-TASK-02 | Sistem wajib menghitung overdue dan days_remaining zona WIB. | M | US-03 |
| FR-GOAL-01 | Sistem wajib menegakkan jenjang parent (quarterly <- annual). | M | US-04 |
| FR-GOAL-02 | Sistem wajib CRUD milestone dengan flag overdue dan auto-stamp completed. | M | US-04 |
| FR-HABIT-01 | Sistem wajib log habit upsert per (habit, tanggal) dan menghitung streak. | M | US-05 |
| FR-FIN-01 | Sistem wajib status budget otomatis (safe/warning/over) dari settings. | M | US-06 |
| FR-FIN-02 | Sistem wajib ringkasan bulan dan annual_cost + days_until langganan. | M | US-06 |
| FR-HEALTH-01 | Sistem wajib health-log upsert per tanggal dan CRUD workout/learning/reading. | S | US-07 |
| FR-REV-01 | Sistem wajib menghitung statistik review saat create/update dan menegakkan period unik. | S | US-08 |
| FR-TRIP-01 | Sistem wajib CRUD trip + itinerary + packing dan actual_cost = SUM biaya. | S | US-09 |
| FR-DEC-01 | Sistem wajib ranking terbobot dan rekomendasi skor tertinggi. | S | US-10 |
| FR-MANAGE-01 | Sistem wajib ETA tabungan, flag garansi aset, days_until dokumen, followup_due kontak. | S | US-11 |
| FR-REM-01 | Sistem wajib next-occurrence dari recurrence dan daftar upcoming. | S | US-12 |
| FR-DATA-01 | ETL wajib menggagalkan run bila quality gate ERROR. | S | US-13 |
| FR-OPS-01 | Sistem wajib backup file rotasi 7 hari dan restore yang menolak bila API jalan. | S | US-14 |

### 3.3 Kinerja

| ID | Kebutuhan | Verifikasi | Bukti |
|---|---|---|---|
| NFR-PERF-01 | `GET /healthz` wajib 200 pada boot normal. | Uji boot runbook | `RUNBOOK.md` repo `-ops` |
| NFR-PERF-02 | Suite QA wajib: 61/61 TC Pass, Newman 75/75, Playwright 26/26, coverage service >= 60% (aktual 83,4%). | CI QA | `data/results.yaml`, `reports/newman.json`, `reports/playwright.json`, `data/bugs.yaml` repo `-qa` |

### 3.4 Atribut Sistem

| ID | Kebutuhan | Verifikasi | Bukti |
|---|---|---|---|
| NFR-SEC-01 | Password wajib bcrypt; JWT wajib secret >= 32 char di produksi. | Review kode | `internal/` repo `lifeos`, `.env.example` |
| NFR-SEC-02 | Secret dan DB (`*.db*`) wajib tidak ter-commit (dukungan: `redact_reports.py`). | CI + review | `tools/redact_reports.py` repo `-qa` |
| NFR-REL-01 | Backup harian + drill restore; postmortem wajib untuk insiden High. | Drill ops | `RUNBOOK.md`, `tickets.yaml` repo `-ops` |
| NFR-MAINT-01 | Skema DB wajib berversi via migrasi goose; tanpa ORM (ADR-001). | Review migrasi | `migrations/00001`-`00015` repo `lifeos` |
| NFR-USA-01 | Web berbahasa Indonesia, tema dark navy, tanpa gradien/shadow/emoji. | Review UI | Repo `lifeos-web` |
| NFR-AVAIL-01 | API wajib menolak start bila migrasi belum applied; restore wajib menolak bila API jalan agar DB tidak korup. | Uji runbook | `RUNBOOK.md` repo `-ops`, guard restore |

## 4. Glosarium

Lihat bagian 1.3.

## 5. Referensi

Lihat bagian 1.4.

## 6. Matriks Traceability

| BRD | PRD | FSD | SRS | Uji (lifeos-qa) |
|---|---|---|---|---|
| BR-01 | US-01 | FS-02/03 | FR-AUTH-01/02 | TC AUTH (6) |
| BR-02 | US-02 | FS-07 | FR-DASH-01 | TC Dashboard (2) |
| BR-03 | US-03 | FS-08-13 | FR-TASK-01/02 | TC Tasks (8) |
| BR-04 | US-04 | FS-14-18 | FR-GOAL-01/02 | TC Goals (7) |
| BR-05 | US-05 | FS-19-23 | FR-HABIT-01 | TC Habits (4) |
| BR-06 | US-06 | FS-24-31,55 | FR-FIN-01/02 | TC Finance (7) |
| BR-07 | US-07 | FS-32-37 | FR-HEALTH-01 | TC HealthLearn (3) |
| BR-08 | US-08 | FS-38-43 | FR-REV-01 | TC Reviews (3) |
| BR-09 | US-09 | FS-44-49 | FR-TRIP-01 | TC Travel (3) |
| BR-10 | US-10 | FS-50-54 | FR-DEC-01 | TC Decisions (3) |
| BR-11 | US-11 | FS-55-67 | FR-MANAGE-01 | TC Assets (3), Contacts (3) |
| BR-12 | US-12 | FS-68-70 | FR-REM-01 | TC Reminders (2), Calendar (3) |
| BR-13 | US-13 | FSD bagian 6 | FR-DATA-01, FR-INT-03 | quality gates |
| BR-14 | US-14 | FSD bagian 7 | FR-OPS-01 | runbook drill, tickets |
| - | - | - | - | TC API (4): Newman, swagger, coverage |

Checklist validasi: semua bagian IEEE 830 terisi; semua kebutuhan ber-ID unik dan terverifikasi; istilah terdefinisi; matriks lengkap. Gap tercatat: swagger tak mendokumentasikan 5 sub-path itinerary/packing/options (FSD bagian 2).
