# PRD - LifeOS

| | |
|---|---|
| Versi | 1.0.0 (2026-09-18) |
| Acuan | BRD v1.0.0 (`BR-01`-`BR-14`) |

## 1. Persona

Satu persona: **Pemilik** - individu yang mengelola hidupnya (tugas, uang, kesehatan, refleksi) lewat dashboard harian. Merangkap operator (backup/restore).

## 2. Fitur

### F-AUTH - Akun (BR-01)

- US-01: Sebagai pemilik, saya dapat register sekali dan login harian.
  - AC: register pertama 201; register kedua -> 403 `single_user_only`; login salah -> 401; token JWT 24 jam; token rusak -> auto-logout web.

### F-DASH - Dashboard (BR-02)

- US-02: Sebagai pemilik, saya melihat 5 KPI + pengingat 7 hari dalam 1 panggilan.
  - AC: tugas due/overdue/total, habit done/total, net cashflow IDR, goals aktif/at-risk, subs jatuh tempo 14 hari, reminders T-days.

### F-TASK - Tugas & Proyek (BR-03)

- US-03: Sebagai pemilik, saya dapat quick-add inbox, filter (Semua/Hari ini/Seminggu/status), DONE, hapus (konfirmasi), dan lihat progres proyek.
  - AC: status enum 6 nilai; completed auto-stamp `completed_at`; overdue = due < hari ini dan bukan completed/cancelled; PUT full-replace (field tak dikirim dikosongkan).

### F-GOAL - Goals & Milestone (BR-04)

- US-04: Sebagai pemilik, saya dapat menyusun goals annual -> quarterly -> monthly + milestone dengan target date.
  - AC: parent quarterly wajib annual; milestone overdue flag; progres = current/target (cap 0-100); update inline + DONE.

### F-HABIT - Habit (BR-05)

- US-05: Sebagai pemilik, saya dapat centang habit harian dan melihat streak + heatmap 30 hari.
  - AC: frekuensi daily/weekly; log upsert per (habit, tanggal); streak dihitung berurutan.

### F-FIN - Keuangan (BR-06)

- US-06: Sebagai pemilik, saya dapat mencatat transaksi, memantau budget dengan status otomatis, dan melihat renewal langganan.
  - AC: budget status safe (< warn) / warning (< 100) / over (>= 100) dari settings; subs annual_cost + days_until; summary bulan income/expense/net (seed Sep: in 10jt/out 2665999/net 7334001).

### F-HEALTH - Sehat & Belajar (BR-07)

- US-07: Sebagai pemilik, saya dapat mencatat health harian (sleep/mood/weight), workout, progres learning, dan status bacaan.
  - AC: energy/mood skala 1-5; health upsert per tanggal; learning progress 0-100; reading rating 1-5.

### F-REVIEW - Review (BR-08)

- US-08: Sebagai pemilik, saya dapat menulis review mingguan/bulanan/tahunan dengan statistik otomatis.
  - AC: stats (tasks due/done/overdue, habit checks, income/expense, goals done/active) dihitung API saat create/update; period unik.

### F-TRAVEL - Travel (BR-09)

- US-09: Sebagai pemilik, saya dapat merencanakan trip (itinerary + packing) dan memantau budget vs aktual.
  - AC: actual_cost = SUM biaya itinerary; item itinerary/packing bisa toggle/hapus.

### F-DECIDE - Keputusan (BR-10)

- US-10: Sebagai pemilik, saya dapat membandingkan opsi dengan skor berbobot.
  - AC: ranking = SUM(weight x score); rekomendasi = skor tertinggi; nilai upsert per (opsi, kriteria).

### F-MANAGE - Aset & Relasi (BR-11)

- US-11: Sebagai pemilik, saya dapat memantau tabungan (progres + ETA), aset (flag garansi), wishlist, dokumen (days_until), dan kontak (follow-up).
  - AC: ETA bulan = sisa/monthly_contribution; followup_due = last_contact + followup_days; tombol SAPA = touch hari ini. Halaman Aset read-only (tanpa create).

### F-REMIND - Pengingat & Kalender (BR-12)

- US-12: Sebagai pemilik, saya dapat mencatat pengingat berulang dan melihat agregat kalender bulanan.
  - AC: recurrence none/daily/weekly/monthly/yearly -> next occurrence; kalender frontend-only (simbol tugas/pengingat/subs/milestone, navigasi bulan).

### F-DATA - Analitik (BR-13)

- US-13: Sebagai pemilik, saya mendapat marts parquet + 4 grafik + laporan quality JSON tiap Senin 06:00.
  - AC: quality gate menggagalkan run (tabel kosong, orphan, amount <= 0, tanggal rusak, duplikat); timer systemd + cron.example; DB pribadi read-only, folder work/ tak di-commit.

### F-OPS - Operasional (BR-14)

- US-14: Sebagai operator, saya punya runbook, SLA, troubleshooting terverifikasi, backup/restore file, dan tiket insiden.
  - AC: backup copy file + rotasi 7; restore menolak bila API jalan; 10 tiket tercatat (9 closed, 1 open).

## 3. Non-Goals (v1.0)

Multi-user, deploy produksi, performance testing, aplikasi mobile, Docker.
