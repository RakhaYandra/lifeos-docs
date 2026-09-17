# FSD - LifeOS

| | |
|---|---|
| Versi | 1.0.0 (2026-09-18) |
| Acuan | PRD v1.0.0 |

## 1. Arsitektur

- Backend Go 1.27/Gin + SQLite (`modernc.org/sqlite`, pure-Go tanpa CGO) + JWT (repo `lifeos`). Tanpa Docker by design; DB file lokal `lifeos.db`.
- Layer: handler (Gin binding) -> service (logika: overdue, streak, utilisasi, ranking, statistik) -> repository (`database/sql` tanpa ORM - ADR-001) -> SQLite. Migrasi goose (15 file 00001-00015).
- Frontend Vite + React 19 + TS + Tailwind v4 (repo `lifeos-web`). Tanpa react-router: state `Page` 13 halaman + Login, pill nav.
- QA: Newman 75 cek + Playwright 26 test + `tools/*.py` (repo `lifeos-qa`).
- Data: ETL Python read-only -> DuckDB parquet + matplotlib PNG + timer mingguan (repo `lifeos-data`).
- Ops: runbook/SLA/tiket + script backup/restore file (repo `lifeos-ops`).
- Env: `PORT=8080`, `DB_PATH=lifeos.db`, `JWT_SECRET` (>= 32 char di prod), `FRONTEND_URL=http://localhost:5174`.
- Auth: single-user; `POST /v1/auth/register` hanya akun pertama (kedua -> 403 `single_user_only`); JWT 24 jam (claim `sub=userID`); selain `/healthz` + auth, semua endpoint wajib Bearer (401 `unauthorized`); CORS echo `Origin == FRONTEND_URL`.
- Konvensi: PUT = full-replace (field tak dikirim dikosongkan); semua data di-scope `user_id`; zona Asia/Jakarta.

## 2. Endpoint API (per `internal/handler/router.go`)

| ID | Method & Path | Fungsi |
|---|---|---|
| FS-01 | `GET /healthz` | Publik. 200 `{"status":"ok"}` |
| FS-02 | `POST /v1/auth/register` | Publik. Akun pertama; validasi email + password min 8, bcrypt |
| FS-03 | `POST /v1/auth/login` | Publik. -> `{token}` JWT 24 jam |
| FS-04 | `GET /v1/me` | Profil sendiri |
| FS-05 | `GET /v1/settings`, `PUT /v1/settings` | Baca/ganti (req: active_year, currency) |
| FS-06 | `GET /v1/life-areas` | 8 area (seed) |
| FS-07 | `GET /v1/dashboard` | Agregat home: tasks due/overdue, habits, finance, goals, subs, reminders |
| FS-08 | `GET,POST /v1/projects` | List + progress / buat (req: name) |
| FS-09 | `GET,PUT,DELETE /v1/projects/:id` | Detail / update / hapus |
| FS-10 | `GET,POST /v1/tasks` | List (?status, ?project_id; overdue/days_remaining) / buat inbox (req: title) |
| FS-11 | `GET /v1/tasks/today` | Due hari ini (WIB) |
| FS-12 | `GET /v1/tasks/week` | Seminggu (?start, default hari ini) |
| FS-13 | `GET,PUT,DELETE /v1/tasks/:id` | Detail / update (completed auto-stamp) / hapus |
| FS-14 | `GET,POST /v1/goals` | List (?level annual/quarterly/monthly, cascade parent) / buat (req: title) |
| FS-15 | `GET,PUT,DELETE /v1/goals/:id` | Detail / update / hapus |
| FS-16 | `GET,POST /v1/milestones` | List (?goal_id, ?project_id, flag overdue) / buat (req: title, target_date) |
| FS-17 | `GET /v1/milestones/upcoming` | Terdekat (?within_days=14) |
| FS-18 | `GET,PUT,DELETE /v1/milestones/:id` | Detail / update (completed auto-stamp) / hapus |
| FS-19 | `GET,POST /v1/habits` | List + streak / buat (req: name) |
| FS-20 | `GET /v1/habits/streaks` | Semua streak |
| FS-21 | `GET,PUT,DELETE /v1/habits/:id` | Detail / update / hapus |
| FS-22 | `POST /v1/habits/:id/log` | Centang (default hari ini WIB; body date/done) |
| FS-23 | `GET /v1/habits/:id/logs` | Riwayat (?from, ?to; heatmap) |
| FS-24 | `GET,POST /v1/transactions` | List (?type, ?category, ?year, ?month) / catat (req: date, type, category, amount) |
| FS-25 | `GET /v1/transactions/summary` | Ringkasan bulan |
| FS-26 | `GET,PUT,DELETE /v1/transactions/:id` | Detail / update / hapus |
| FS-27 | `GET,POST /v1/budgets` | List + aktual otomatis (?year, ?month) / buat (req: year, month, category, amount) |
| FS-28 | `GET,PUT,DELETE /v1/budgets/:id` | Detail / update / hapus |
| FS-29 | `GET,POST /v1/subscriptions` | List + annual_cost + days_until / tambah (req: service, next_billing) |
| FS-30 | `GET /v1/subscriptions/upcoming` | Renewal (?within_days=14) |
| FS-31 | `GET,PUT,DELETE /v1/subscriptions/:id` | Detail / update / hapus |
| FS-32 | `GET,PUT /v1/health-logs` | List (?from, ?to) / upsert per tanggal (req: date) |
| FS-33 | `GET,POST /v1/workouts` | List (?from, ?to) / catat (req: date, type) |
| FS-34 | `GET,POST /v1/learning` | List / tambah (req: topic) |
| FS-35 | `PUT,DELETE /v1/learning/:id` | Update / hapus (tanpa GET detail) |
| FS-36 | `GET,POST /v1/reading` | List / tambah (req: title) |
| FS-37 | `PUT,DELETE /v1/reading/:id` | Update / hapus (tanpa GET detail) |
| FS-38 | `GET,POST /v1/reviews` | List + auto-stat / buat mingguan (req: week_start) |
| FS-39 | `GET,PUT,DELETE /v1/reviews/:id` | Detail / update (hitung ulang stats) / hapus |
| FS-40 | `GET,POST /v1/monthly-reviews` | List / buat (req: period YYYY-MM) |
| FS-41 | `GET,PUT,DELETE /v1/monthly-reviews/:id` | Detail / update / hapus |
| FS-42 | `GET,POST /v1/yearly-reviews` | List / buat (req: period YYYY) |
| FS-43 | `GET,PUT,DELETE /v1/yearly-reviews/:id` | Detail / update / hapus |
| FS-44 | `GET,POST /v1/trips` | List + actual_cost / buat (req: name, start_date, end_date) |
| FS-45 | `GET,PUT,DELETE /v1/trips/:id` | Detail (+itinerary+packing) / update / hapus |
| FS-46 | `POST /v1/trips/:id/itinerary` | Tambah item (req: date, activity) |
| FS-47 | `PUT,DELETE /v1/trips/:id/itinerary/:iid` | Toggle / hapus item |
| FS-48 | `POST /v1/trips/:id/packing` | Tambah item (req: item) |
| FS-49 | `PUT,DELETE /v1/trips/:id/packing/:pid` | Toggle / hapus item |
| FS-50 | `GET,POST /v1/decisions` | List + ranking terbobot / buat (req: title) |
| FS-51 | `GET,DELETE /v1/decisions/:id` | Detail + ranking / hapus (tanpa PUT) |
| FS-52 | `POST /v1/decisions/:id/options` | Tambah opsi (req: name) |
| FS-53 | `DELETE /v1/decisions/:id/options/:oid` | Hapus opsi |
| FS-54 | `POST /v1/decisions/:id/options/:oid/marks` | Nilai opsi upsert per kriteria (req: criterion, score; weight opsional) |
| FS-55 | `GET,POST /v1/savings` | List + progres + ETA / tambah (req: name, target_amount) |
| FS-56 | `PUT,DELETE /v1/savings/:id` | Update / hapus (tanpa GET detail) |
| FS-57 | `GET,POST /v1/assets` | List + flag garansi / tambah (req: name) |
| FS-58 | `DELETE /v1/assets/:id` | Hapus (tanpa PUT/GET detail) |
| FS-59 | `GET,POST /v1/wishlist` | List + progres / tambah (req: item) |
| FS-60 | `DELETE /v1/wishlist/:id` | Hapus |
| FS-61 | `GET,POST /v1/documents` | List + days_until / tambah (req: item, expiry_date) |
| FS-62 | `GET /v1/documents/upcoming` | Hampir kedaluwarsa |
| FS-63 | `DELETE /v1/documents/:id` | Hapus |
| FS-64 | `GET,POST /v1/contacts` | List + followup_due / tambah (req: name) |
| FS-65 | `GET /v1/contacts/followups` | Butuh follow-up |
| FS-66 | `POST /v1/contacts/:id/touch` | Catat kontak hari ini |
| FS-67 | `DELETE /v1/contacts/:id` | Hapus |
| FS-68 | `GET,POST /v1/reminders` | List + next/days_until / tambah (req: title, date) |
| FS-69 | `GET /v1/reminders/upcoming` | Terdekat (?within_days=14) |
| FS-70 | `GET,PUT,DELETE /v1/reminders/:id` | Detail / update / hapus |

Catatan: `api/swagger.yaml` tidak mendokumentasikan `PUT/DELETE itinerary/:iid`, `PUT/DELETE packing/:pid`, `DELETE options/:oid` (gap dokumentasi, fungsi tetap berjalan).

## 3. Model Data (SQLite, goose 00001-00015)

- `users(id, email UNIQUE, password_hash, created_at)`; `settings(user_id PK, active_year, currency, budget_warn_pct, goal_warn_pct)`; `life_areas(id, name UNIQUE)` (8 seed).
- `projects(id, user_id, name, life_area_id NULL, goal_id, status, start/end/completed_at, notes)`; `tasks(id, user_id, project_id SET NULL, goal_id, life_area_id, title, status, priority, due_date, completed_at, effort_est/actual)`.
- `goals(id, user_id, level, parent_id self-ref, life_area_id, title, metric, target/current_value, status, target_date)`; `milestones(id, user_id, goal_id, project_id, title, target_date, status, completed_at, notes)`.
- `habits(id, user_id, name, frequency, target_per_week, start_date, active)` + `habit_logs(habit_id, date PK komposit, done)`.
- `transactions(id, user_id, date, type, category, description, amount > 0, account, recurring)`; `budgets(id, user_id, year, month, category, amount > 0, UNIQUE per kategori-bulan)`; `subscriptions(id, user_id, service, category, cost >= 0, frequency, next_billing, active)`.
- `health_logs(id, user_id, date UNIQUE, weight, sleep, water, energy, mood, notes)`; `workouts(id, user_id, date, type, duration_min, intensity, calories, notes)`; `learning_entries(...)`; `reading_entries(...)`.
- `reviews/monthly_reviews/yearly_reviews(id, user_id, week_start/period UNIQUE, stats JSON, wins, challenges, lessons, next_focus)`.
- `reminders(id, user_id, title, date, recurrence, notes)`.
- `trips` + `itinerary_items` + `packing_items` (CASCADE); `decisions` + `decision_options` + `decision_marks` (CASCADE, UNIQUE per opsi+kriteria).
- `savings_goals`, `assets`, `wishlist`, `documents`, `contacts`.
- Relasi: users 1-N semua; goals self-cascade; FK via PRAGMA.

## 4. Aturan Bisnis

1. Tasks/milestones completed -> auto-stamp `completed_at`; overdue = due < hari ini dan bukan completed/cancelled.
2. Project progress = done/total (cap 0-100); goal progress = current/target (cap 0-100).
3. Budget actual = SUM expense kategori-bulan; util = actual/budget x 100; status safe/warning/over dari settings.
4. Habit streak berurutan; log upsert per (habit, tanggal); heatmap 30 hari.
5. Subs annual_cost + days_until; savings progres + ETA bulan = sisa/monthly_contribution.
6. Reviews stats JSON dihitung API saat create/update.
7. Reminders next-occurrence dari recurrence; upcoming <= 7/14 hari.
8. Trips actual_cost = SUM biaya itinerary; decisions ranking = SUM(weight x score), rekomendasi = max.
9. Assets flag garansi dari warranty_end; documents days_until + upcoming via reminder_days; contacts followup_due.
10. Enum validasi per modul (task 6 status, goal level/status, trx type, dsb - lihat FSD riset/appendiks kode).

## 5. Web UI (13 halaman + Login)

Dashboard (5 KPI + reminders 7d), Tasks (toggle + filter + quick-add + DONE/DEL + progres proyek), Goals (tab level + parent picker + UPDATE inline + milestone), Habits (streak + CENTANG + heatmap), Finance (input bulan + cashflow + budget util + subs + tabungan), Health (health today + workout + learning + reading), Calendar (grid bulan frontend-only + legenda), Travel (pill trip + itinerary + packing), Decisions (pill + ranking + form nilai), Assets (read-only: aset + wishlist + dokumen), Contacts (FOLLOW-UP + SAPA/touch), Reviews (tab period + stats + WINS/BLOCK/NEXT), Reminders (recurrence + DEL), Login (hero + form, prefill seed). Token `localStorage["lifeos_token"]`; 401 -> auto-logout per halaman. Base: `VITE_API_URL` default `http://localhost:8080`.

## 6. ETL / Analitik

Alur `run.py`: extract 9 tabel (read-only `mode=ro`) -> quality gates (ERROR gagalkan run: tabel kosong, orphan, amount <= 0, tanggal rusak, duplikat id; WARN: status di luar enum) -> 5 marts (`spending_daily`, `cashflow_monthly`, `budget_health`, `habit_rates`, `task_throughput`) -> parquet DuckDB -> 4 PNG (spending_by_category, cashflow_trend, habit_rates, task_throughput) -> `work/quality_report.json` + insight. Jadwal: systemd timer Senin 06:00 + cron.example. `run.sh` bootstrap `work/seed.db` bila absen; hormati `LIFEOS_DB` untuk data pribadi.

## 7. Operasional

Tanpa Docker. Run: `goose up` + seed + `go run ./cmd/api` (:8080) + `npm run dev` (:5174). Backup: copy `lifeos.db` -> `backups/lifeos-<TS>.db`, rotasi 7. Restore: guard tolak bila API jalan. Seed: 1 user (`aku@lifeos.local` / `Rahasia123`) + 7 goals + 25 tasks + 5 habits/80 logs + 32 trx + 4 budgets + 4 subs + health/learning/review/reminder/trip/decision/savings/asset/wishlist/document/contact fiktif.
