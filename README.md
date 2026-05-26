# ProAV Team Scheduler

A lightweight, browser-based team scheduling tool built for ProAV. It provides a shared calendar for tracking jobs, assignments, and availability across the team — accessible on desktop and mobile.

---

## Access

**Live URL:** `https://inthenomansland.github.io/Scheduler`

Sign in with your ProAV email address and password. Contact Ashton if you need an account set up.

---

## Tech Stack

| Layer | Technology |
|---|---|
| Frontend | HTML, CSS, JavaScript (no framework) |
| Database | [Supabase](https://supabase.com) (PostgreSQL) |
| Hosting | [GitHub Pages](https://github.com/inthenomansland/Scheduler) |
| Auth | Supabase Authentication |

---

## Features

**Calendar**
- Monthly calendar view on desktop
- Mobile view with dot indicators per day and a day detail panel (Apple Calendar style)
- Mobile month overview with job bars
- Swipe left/right on mobile to change month
- Bank holidays (England) highlighted automatically (hardcoded 2024–2027)
- Weekends greyed out — job bars do not display on Saturdays or Sundays even if a job spans them

**Jobs**
- Add, edit and delete jobs with: job name, SEQF number, category, assigned person, project manager, start/end date, and notes
- Four job categories: Offsite work (yellow), Onsite work (blue), Witness test (green), Annual leave (red)
- Witness tests are excluded from conflict detection
- Jobs with notes show a 📋 indicator on the calendar bar
- Unassigned jobs show a ⚠ indicator

**Conflict detection**
- Automatically flags double assignments in the panel below the calendar
- Only checks current and future jobs — past conflicts are ignored
- Witness tests are excluded from conflict checks

**Filters**
- Filter by team member and/or job type simultaneously
- Active filters highlighted; clear all with one tap

**Search**
- Search by job name, SEQF number, project manager, assigned person, or notes
- Click a result to jump straight to that month with the job highlighted

**Team full indicator**
- When Ashton, Chris, Ross, Aidan and Alain all have jobs on the same day, that day gets a red outline

**Access levels**
- **Admin** — can add, edit and delete jobs
- **Read-only** — can view all jobs and open job detail cards but cannot make changes

---

## Team Members

Current team in the app:

- Ashton (admin)
- Scott
- Chris
- Ross
- Aidan
- Alain

Scott is excluded from the "team full" indicator by design.

---

## User Roles

### Granting admin access

Run this in **Supabase → SQL Editor**, replacing the email:

```sql
update auth.users 
set raw_user_meta_data = raw_user_meta_data || '{"role":"admin"}'::jsonb
where email = 'their@email.com';
```

### Read-only access

Any user invited via Supabase Authentication who does not have the admin role will automatically get read-only access. No extra steps needed.

---

## Adding a New Team Member

**Step 1 — Invite them in Supabase**
1. Go to **Supabase → Authentication → Users → Invite user**
2. Enter their email address
3. They'll receive an email to set their password

**Step 2 — Add their name to the app**
1. Open `index.html` in GitHub
2. Find the `PEOPLE` array near the top of the `<script>` section:
```js
const PEOPLE=['Unassigned','Ashton','Scott','Chris','Ross','Aidan','Alain'];
```
3. Add their name to the list
4. If they should be included in the "team full" check, also add them to:
```js
const CONFLICT_CHECK_PEOPLE=['Ashton','Chris','Ross','Aidan','Alain'];
```
5. Commit the change — live within 60 seconds

---

## Updating the App

For small changes (names, colours, categories):
1. Go to the [GitHub repository](https://github.com/inthenomansland/Scheduler)
2. Click `index.html`
3. Click the **pencil icon** to edit
4. Make your change and click **Commit changes**
5. Live within ~60 seconds

For larger changes or new features, bring the current `index.html` code into a Claude conversation and describe what you need.

---

## Database Structure

**Supabase project:** `https://xtuexrktnzrwadgvqeaw.supabase.co`

### `jobs` table

| Column | Type | Notes |
|---|---|---|
| `id` | uuid | Auto-generated primary key |
| `name` | text | Job name — required |
| `cat` | text | Category ID: `work`, `onsite`, `witness`, `leave` |
| `person` | text | Assigned team member |
| `start_date` | date | Job start date |
| `end_date` | date | Job end date |
| `notes` | text | Optional notes |
| `seqf` | text | Optional SEQF reference number |
| `pm` | text | Optional project manager name |
| `created_at` | timestamp | Auto-set on creation |

### Setting up the database from scratch

Run in **Supabase → SQL Editor**:

```sql
create table jobs (
  id uuid default gen_random_uuid() primary key,
  name text not null,
  cat text not null,
  person text not null,
  start_date date not null,
  end_date date not null,
  created_at timestamp default now()
);

alter table jobs enable row level security;

create policy "Allow all for authenticated users"
on jobs for all
using (auth.role() = 'authenticated');

alter table jobs add column notes text;
alter table jobs add column seqf text;
alter table jobs add column pm text;
```

---

## Known Limitations

- **Bank holidays are hardcoded** to 2024–2027. These will need updating manually in the `BANK_HOLIDAYS` set inside `index.html` when 2028 dates are published by the UK government.
- **Data is not real-time** — if two admins are editing simultaneously, one may overwrite the other. Refresh the page to get the latest data.
- **localStorage is not used** — all data lives in Supabase. Clearing your browser data will not affect the scheduler.
- **No password reset UI** — password resets must be triggered via Supabase Authentication or by an admin re-inviting the user.

---

## Built With

Developed iteratively using [Claude](https://claude.ai) by Anthropic.
