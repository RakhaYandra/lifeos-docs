# BRD - LifeOS

| | |
|---|---|
| Versi | 1.0.0 (2026-09-18) |
| Sistem | LifeOS - personal operating system |
| Bahasa | Indonesia |

## 1. Latar Belakang

Produktivitas pribadi tersebar di banyak aplikasi (to-do, catatan keuangan, habit tracker, jurnal) sehingga tidak ada gambaran utuh. LifeOS menyatukan 13 area hidup (tugas, goals, habits, keuangan, kesehatan, belajar, travel, keputusan, aset, relasi, review, pengingat, kalender) dalam satu dashboard single-user.

## 2. Masalah Bisnis

1. Data hidup tercecer di banyak aplikasi sehingga pemilik tidak punya gambaran utuh saat merencanakan hari.
2. Tugas, goals, dan milestone tidak tertaut sehingga progres sulit diukur.
3. Arus kas, budget, dan langganan tercatat terpisah sehingga kebocoran sulit ketahuan.
4. Refleksi (mingguan/bulanan/tahunan) tidak berbasis data karena statistik tersebar.

## 3. Tujuan Bisnis

| ID | Tujuan |
|---|---|
| BO-01 | Satu dashboard harian yang layak dibuka tiap pagi (telemetri hidup). |
| BO-02 | Tugas, goals berjenjang, dan milestone saling tertaut dan terukur. |
| BO-03 | Arus kas, budget, langganan, tabungan tercatat dengan status otomatis. |
| BO-04 | Konsistensi habit terpantau via streak dan heatmap 30 hari. |
| BO-05 | Refleksi mingguan/bulanan/tahunan dengan statistik otomatis. |
| BO-06 | Keputusan besar didukung ranking berbobot yang terdokumentasi. |

## 4. Stakeholder

| Peran | Tanggung Jawab | Kepentingan |
|---|---|---|
| Pemilik (single-user) | Satu-satunya pengguna; kelola 13 area hidup | Satu dashboard utuh miliknya |
| Operator (pemilik merangkap) | Backup, restore, update, tiket insiden | Data aman, app jalan |

Tidak ada peran admin/manajer/staf: single-user by design.

## 5. Ruang Lingkup

**Masuk (IN):** akun pemilik tunggal; dashboard agregat; CRUD tasks, projects, goals + milestones, habits + logs, transactions + budgets + subscriptions, health-logs + workouts, learning + reading, reviews (weekly/monthly/yearly), trips + itinerary + packing, decisions + options + marks, savings + assets + wishlist + documents + contacts, reminders; web 13 halaman + login; analitik offline (marts + charts + quality gates + jadwal mingguan); runbook/SLA/tiket operasional.

**Keluar (OUT):** multi-user, deploy/infra produksi, performance testing, mobile app, Docker (tanpa Docker by design - data lokal).

## 6. Kebutuhan Bisnis

| ID | Kebutuhan | Prioritas |
|---|---|---|
| BR-01 | Akun tunggal pemilik dengan sesi login 24 jam | Must |
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
| BR-13 | Analitik offline dengan quality gates + jadwal mingguan | Should |
| BR-14 | Runbook, SLA, dan tiket insiden terdokumentasi | Should |

## 7. Aturan Bisnis

1. Hanya 1 akun pemilik; pendaftaran kedua wajib ditolak.
2. Tugas/milestone selesai otomatis mencatat waktu selesai; overdue = lewat due date dan belum selesai/dibatalkan.
3. PUT bersifat full-replace: field yang tidak dikirim dikosongkan.

## 8. Risiko

| Risiko | Dampak | Kemungkinan | Mitigasi |
|---|---|---|---|
| DB file lokal hilang/rusak | Tinggi | Rendah | Backup file harian retensi 7 hari; restore guard menolak bila API jalan |
| Lupa password / token kedaluwarsa | Sedang | Sedang | Login ulang; token 24 jam by design; secret minimal 32 char |
| PUT full-replace menghapus field | Sedang | Sedang | By design dan terdokumentasi; UI selalu kirim objek penuh |
| Seed tanggal tetap (overdue/streak geser) | Rendah | Tinggi | QA melampirkan tanggal run; data dihitung dinamis dari hari berjalan |
| DB terkunci saat backup | Sedang | Rendah | Backup = copy file; restore wajib matikan API dulu |

## 9. SLA (ringkas, detail di repo `-ops`)

| Severity | Contoh | Respons | Resolusi |
|---|---|---|---|
| High | App mati total | 1 jam | 1 hari |
| Medium | Sebagian fitur mati | 4 jam | 3 hari |
| Low | Kosmetik | 2 hari | 2 pekan |

Eskalasi naik 1 level bila respons terlampaui; postmortem wajib untuk High; ada jam kerja.

## 10. Kriteria Sukses

| ID | Kriteria |
|---|---|
| SC-01 | 61/61 TC QA Pass, Newman 75/75, Playwright 26/26, coverage service >= 60% (aktual 83,4%). |
| SC-02 | 0 bug Critical/High terbuka (4 bug Medium/Low fixed). |
| SC-03 | CI hijau di 5 repo lifeos + docs terbit v1.0.0. |
