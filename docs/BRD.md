# BRD - LifeOS

| | |
|---|---|
| Versi | 1.0.0 (2026-09-18) |
| Sistem | LifeOS - personal operating system |
| Bahasa | Indonesia |

## 1. Latar Belakang

Produktivitas pribadi tersebar di banyak aplikasi (to-do, catatan keuangan, habit tracker, jurnal) sehingga tidak ada gambaran utuh. LifeOS menyatukan 13 area hidup (tugas, goals, habits, keuangan, kesehatan, belajar, travel, keputusan, aset, relasi, review, pengingat, kalender) dalam satu dashboard single-user: satu database SQLite lokal, satu API, satu web.

## 2. Tujuan Bisnis

1. Satu dashboard harian yang layak dibuka tiap pagi (telemetri hidup).
2. Tugas, goals berjenjang, dan milestone saling tertaut dan terukur.
3. Arus kas, budget, langganan, tabungan tercatat dengan status otomatis.
4. Konsistensi habit terpantau via streak dan heatmap 30 hari.
5. Refleksi mingguan/bulanan/tahunan dengan statistik otomatis.
6. Keputusan besar didukung ranking berbobot yang terdokumentasi.

## 3. Stakeholder

| Peran | Kepentingan |
|---|---|
| Pemilik (single-user) | Satu-satunya pengguna; semua data miliknya |
| Operator (pemilik merangkap) | Backup, restore, update, tiket insiden |

Tidak ada peran admin/manajer/staf: single-user by design.

## 4. Ruang Lingkup

**Masuk (IN):** autentikasi JWT single-user; dashboard agregat; CRUD tasks, projects, goals + milestones, habits + logs, transactions + budgets + subscriptions, health-logs + workouts, learning + reading, reviews (weekly/monthly/yearly), trips + itinerary + packing, decisions + options + marks, savings + assets + wishlist + documents + contacts, reminders; web 13 halaman + login; ETL analitik (marts + charts + quality gates + jadwal mingguan); runbook/SLA/tiket operasional.

**Keluar (OUT):** multi-user, deploy/infra produksi, performance testing, mobile app, Docker (tanpa Docker by design - data lokal SQLite).

## 5. Kebutuhan Bisnis

| ID | Kebutuhan | Prioritas |
|---|---|---|
| BR-01 | Akun tunggal pemilik dengan login JWT 24 jam | Must |
| BR-02 | Dashboard agregat harian (tugas, habit, kas, goals, subs, pengingat) | Must |
| BR-03 | Tugas dengan status, prioritas, due date, overdue, dan tautan proyek/goal | Must |
| BR-04 | Goals berjenjang annual -> quarterly -> monthly + milestones | Must |
| BR-05 | Habit harian/mingguan dengan streak dan heatmap | Must |
| BR-06 | Transaksi, budget bulanan dengan status otomatis, dan langganan dengan pengingat renewal | Must |
| BR-07 | Health log harian + workout + learning/reading tracker | Should |
| BR-08 | Review mingguan/bulanan/tahunan dengan statistik otomatis | Should |
| BR-09 | Trip dengan itinerary, packing, dan biaya aktual | Should |
| BR-10 | Keputusan dengan opsi dan penilaian berbobot | Should |
| BR-11 | Tabungan, aset, wishlist, dokumen, kontak + follow-up | Should |
| BR-12 | Pengingat berulang dengan occurrence berikutnya | Should |
| BR-13 | Analitik offline (parquet + grafik) dengan quality gates + jadwal mingguan | Should |
| BR-14 | Runbook, SLA, dan tiket insiden terdokumentasi | Should |

## 6. Risiko

| Risiko | Mitigasi |
|---|---|
| DB SQLite hilang/rusak (file lokal) | Backup file harian retensi 7 hari; restore guard menolak bila API jalan |
| Lupa password / token kedaluwarsa | Login ulang; JWT 24 jam by design; secret >= 32 char |
| PUT full-replace menghapus field | By design dan terdokumentasi; UI selalu kirim objek penuh |
| Seed tanggal tetap (overdue/streak geser) | QA melampirkan tanggal run; data dihitung dinamis dari hari berjalan |
| SQLite locked saat backup | Backup = copy file; restore wajib matikan API dulu |

## 7. SLA (ringkas, detail di repo `-ops`)

| Severity | Contoh | Respons | Resolusi |
|---|---|---|---|
| High | App mati total | 1 jam | 1 hari |
| Medium | Sebagian fitur mati | 4 jam | 3 hari |
| Low | Kosmetik | 2 hari | 2 pekan |

Eskalasi naik 1 level bila respons terlampaui; postmortem wajib untuk High; ada jam kerja.

## 8. Kriteria Sukses

1. 61/61 TC QA Pass, Newman 75/75, Playwright 26/26, coverage service >= 60% (aktual 83,4%).
2. 0 bug Critical/High terbuka (4 bug Medium/Low fixed).
3. CI hijau di 5 repo lifeos + docs terbit v1.0.0.
