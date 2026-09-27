# Swarsanskar Music Academy – Web Portal

Student + Admin portal for **Swarsanskar Music Academy** (Vasai).  
Handles ABGMV exam form collection, documents, fees, help desk, contact messages, and admin approval — backed by **Supabase**.

**Live (example):** `https://swarsanskar.vercel.app`  
**Stack:** Static HTML/CSS/JS · Supabase (Database + Storage) · GitHub / Vercel hosting

---

## Features

### Public site
- Home, Courses, About, Vision, Contact, Help
- Contact form → saved to database → Admin can Call / WhatsApp / Email
- Help Desk → unique **Help ID** (`HLP-YYYY-XXXX`) → track status + admin reply

### Student portal (`student.html`)
- Login with **Swar ID** + password
- Sidebar: **Dashboard** · **Exam Form**
- Exam form: name, DOB (auto age), course level, exam type (Regular / Direct)
- Document uploads: passport photo, birth proof, optional marksheet
- UPI QR fee payment + payment screenshot
- Fees differ by **Regular** vs **Direct** (editable in code)
- Data + files saved to Supabase
- **Download Receipt (PDF)** — professional receipt with logo

### Admin panel (`panel.html`)
- Sidebar: Dashboard · Users · Exam Form Approval · Help Queries · Contact Messages
- View students, verify exam forms, open uploaded documents
- Reply to Help queries
- Contact messages with Call / WhatsApp / Email shortcuts

### Auth (`Sign.html`)
- Student **Sign Up** → generates unique **Swar ID** (`SMA-YYYY-XXXX`)
- Student / Admin **Sign In**
- Passwords stored in DB (plain for simple academy setup; can be hashed later)

---

## Project files

| File | Purpose |
|------|---------|
| `index.html` | Home |
| `cat.html` | Courses |
| `about.html` | About |
| `vision.html` | Vision |
| `cont.html` | Contact form |
| `help.html` | Help Desk |
| `Sign.html` | Sign in / Sign up |
| `student.html` | Student portal |
| `panel.html` | Admin panel |
| `theme.css` | Shared dark/gold theme |
| `logo.jpeg` | Academy logo |
| `my-qr.jpeg` | UPI QR for exam fees |

---

## Supabase setup

### 1. Create project
1. Go to [https://supabase.com](https://supabase.com) → New project  
2. Copy **Project URL** and **Publishable (anon) key**  
3. Put them in `Sign.html`, `student.html`, `panel.html`, `help.html`, `cont.html`:

```js
const SUPABASE_URL = "https://YOUR_PROJECT.supabase.co";
const SUPABASE_KEY = "sb_publishable_... or eyJ...";
```

### 2. Tables (SQL Editor → Run)

#### Users
```sql
create table if not exists public.users (
  id uuid primary key default gen_random_uuid(),
  swar_id text unique not null,
  first_name text not null,
  last_name text not null,
  email text,
  phone text,
  course text,
  password_hash text not null,
  role text not null default 'student' check (role in ('student', 'admin')),
  is_approved boolean default true,
  created_at timestamptz default now()
);

alter table public.users enable row level security;
create policy "Allow all on users" on public.users for all using (true) with check (true);
```

#### Admin account (change password after first login if needed)
```sql
insert into public.users (swar_id, first_name, last_name, email, password_hash, role, is_approved)
values ('ADMIN', 'Admin', 'Swarsanskar', 'admin@swarsanskar.local', 'SWARSHREYAS', 'admin', true)
on conflict (swar_id) do nothing;
```

#### Exam forms
```sql
create table if not exists public.exam_forms (
  id uuid primary key default gen_random_uuid(),
  swar_id text not null,
  full_name text,
  dob date,
  age int,
  course_level text,
  exam_type text,
  fee_amount numeric,
  photo_path text,
  birth_proof_path text,
  marksheet_path text,
  payment_screenshot_path text,
  payment_confirmed boolean default false,
  status text default 'submitted' check (status in ('pending', 'submitted', 'verified')),
  created_at timestamptz default now(),
  updated_at timestamptz default now()
);

alter table public.exam_forms enable row level security;
create policy "Allow all on exam_forms" on public.exam_forms for all using (true) with check (true);
```

If `exam_type` column is missing on an old table:
```sql
alter table public.exam_forms add column if not exists exam_type text;
```

#### Help queries
```sql
create table if not exists public.help_queries (
  id uuid primary key default gen_random_uuid(),
  help_id text unique not null,
  swar_id text,
  name text,
  message text not null,
  status text not null default 'open' check (status in ('open', 'replied', 'closed')),
  admin_reply text,
  created_at timestamptz default now(),
  replied_at timestamptz
);

alter table public.help_queries enable row level security;
create policy "Allow all on help_queries" on public.help_queries for all using (true) with check (true);
```

#### Contact messages
```sql
create table if not exists public.contact_messages (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  contact text not null,
  message text not null,
  status text not null default 'new' check (status in ('new', 'read')),
  created_at timestamptz default now()
);

alter table public.contact_messages enable row level security;
create policy "Allow all on contact_messages" on public.contact_messages for all using (true) with check (true);
```

### 3. Storage bucket
1. Storage → New bucket → name: **`documents`**  
2. Set **Public** if students/admin should open file links easily  
3. Policies (SQL Editor):

```sql
create policy "Allow public uploads" on storage.objects
  for insert with check (bucket_id = 'documents');
create policy "Allow public read" on storage.objects
  for select using (bucket_id = 'documents');
create policy "Allow public update" on storage.objects
  for update using (bucket_id = 'documents');
```

---

## Local run

1. Keep all HTML/CSS/images in one folder  
2. Open `index.html` or `Sign.html` in a browser  
3. For full API testing, prefer a local static server (optional):

```bash
npx serve .
```

**Note:** Opening via `file://` can block logo on PDF receipt sometimes; hosting (Vercel) is more reliable.

---

## Default admin login

| Field | Value |
|-------|--------|
| Swar ID | `ADMIN` |
| Password | `SWARSHREYAS` |

Change this in Supabase Table Editor after go-live.

---

## Exam fees (edit in `student.html`)

```js
const FEE_MAP = {
  Regular: {
    "Prarambhik": 300,
    "Praveshika Pratham": 400,
    // ...
  },
  Direct: {
    "Prarambhik": 450,
    // ...
  }
};
```

Update numbers anytime and redeploy.

---

## Deploy (Vercel / GitHub Pages)

### Vercel
1. Push folder to GitHub  
2. Import project in Vercel  
3. Deploy (static site, no build needed)  
4. Optional: add custom domain later  

### GitHub Pages
1. Repo → Settings → Pages → branch `main` / root  
2. Site URL: `https://USERNAME.github.io/REPO/`  

Ensure relative paths (`theme.css`, `logo.jpeg`, page links) stay correct.

---

## User flows

### Student
1. Sign Up → note **Swar ID**  
2. Sign In → Student portal  
3. Dashboard → Exam Form  
4. Fill details · upload docs · pay via QR · upload screenshot · Save  
5. Download PDF receipt  

### Admin
1. Sign In as `ADMIN`  
2. Users → registered students  
3. Exam Form Approval → View docs → Mark Verified  
4. Help Queries → Reply  
5. Contact Messages → Call / WhatsApp / Email  

---

## Security notes (current simple setup)

- RLS policies are open (`allow all`) for easy academy use  
- Passwords are stored as plain text in `password_hash` column  
- Publishable key is safe in frontend **only if** RLS is correct  

**Before wide public use, consider:**
- Password hashing (e.g. bcrypt via Edge Function)
- Stricter RLS (students only see own rows)
- Rate limits / CAPTCHA on public forms

---

## Troubleshooting

| Issue | Fix |
|-------|-----|
| `Identifier 'supabase' has already been declared` | Use `var sb = window.supabase.createClient(...)` — do not redeclare `supabase` |
| Storage RLS error on upload | Add storage policies for bucket `documents` |
| Login fails | Check `users` table + correct Project URL / key |
| Receipt logo missing | Ensure `logo.jpeg` is deployed next to `student.html` |
| Marathi text broken in PDF | jsPDF default fonts don’t support Devanagari; use English tagline |

---

## Roadmap ideas

- [ ] Online **Lessons** section (videos + notes by level)
- [ ] Form status lock after admin verifies
- [ ] Email / WhatsApp notify on form submit
- [ ] Custom domain (`swarsanskar.com`)
- [ ] Stronger auth + password hashing

---

## Contact (academy)

- **Email:** swarsankarmusic@gmail.com  
- **Ranjit Gaonkar:** +91 98218 66241  
- **Shreyas Gaonkar:** +91 91728 44879  
- **Address:** Swarsanskar, R/104, Godavari Apartment, Umelman, Vasai – 401202  

---

Built for Swarsanskar Music Academy — *From notes to values.*
