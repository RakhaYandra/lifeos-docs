# LifeOS Docs

[![docs](https://github.com/RakhaYandra/lifeos-docs/actions/workflows/docs-to-pdf.yml/badge.svg)](https://github.com/RakhaYandra/lifeos-docs/releases)

> Ekosistem: [api](https://github.com/RakhaYandra/lifeos) · [web](https://github.com/RakhaYandra/lifeos-web) · [docs](https://github.com/RakhaYandra/lifeos-docs/releases) · [qa](https://github.com/RakhaYandra/lifeos-qa) · [data](https://github.com/RakhaYandra/lifeos-data) · [ops](https://github.com/RakhaYandra/lifeos-ops)

Dokumentasi resmi sistem **LifeOS** (Bahasa Indonesia).

**LifeOS** adalah personal operating system single-user: 13 area hidup (tugas, goals, habits, keuangan, kesehatan, belajar, travel, keputusan, aset, relasi, review, pengingat, kalender) dalam satu dashboard, satu API, satu database file lokal. Pengguna sekaligus operator: Pemilik. Repo ini adalah sumber dokumen requirements dan desain; kode sumber ada di repo aplikasi (lihat Sumber Fakta).

| Dokumen | Isi |
|---|---|
| [docs/BRD.md](docs/BRD.md) | Business Requirements Document — kebutuhan bisnis |
| [docs/PRD.md](docs/PRD.md) | Product Requirements Document — fitur & acceptance criteria |
| [docs/FSD.md](docs/FSD.md) | Functional Specification Document — spesifikasi fungsi/teknis |
| [docs/SRS.md](docs/SRS.md) | Software Requirements Specification (IEEE 830) |

Sumber fakta (sumber kebenaran per artefak):

| Repo | Artefak yang dirujuk |
|---|---|
| `lifeos` | `api/swagger.yaml`, `migrations/00001`-`00015`, `docs/ADR-001-sqlite-no-orm` |
| `lifeos-web` | 13 halaman + Login (state `Page`, pill nav) |
| `lifeos-qa` | `data/testcases.yaml` (61 TC), `data/results.yaml`, `data/bugs.yaml`, Newman, Playwright |
| `lifeos-data` | ETL 5 marts (spending_daily, cashflow_monthly, budget_health, habit_rates, task_throughput), timer Senin 06:00 |
| `lifeos-ops` | `RUNBOOK.md`, `SLA.md`, `TROUBLESHOOTING.md`, `tickets.yaml` |

## PDF

Tiap tag `v*` memicu workflow `docs-to-pdf` -> PDF di-upload sebagai **Release asset**.
Unduh versi formal di halaman [Releases](../../releases).

## Versioning

| Versi | Tanggal | Isi |
|---|---|---|
| v1.0.0 | 2026-09-18 | Rilis awal: BRD, PRD, FSD, SRS + PDF |
| v1.1.0 | 2026-09-20 | Penambahan Guideline v1.0: ID BO/SC, flows PRD, spec pointer FSD, ERD, NFR verifikasi |
