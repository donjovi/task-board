# Task Board

Shared multi-user task board. One HTML file, hosted on GitHub Pages, backed by a free Supabase database. Everyone with the link sees the same board, live.

## Features

- Add tasks, mark complete (tracks who added and who completed)
- Color-coded categories, created on the fly, used as filter tabs
- Priority levels (Low / Normal / High / Urgent)
- Optional due dates with overdue highlighting
- Optional notes per task
- Live sync — changes appear for everyone without refreshing
- No passwords: users just pick a display name once

## One-time setup (about 5 minutes)

### 1. Create the free database

1. Go to [supabase.com](https://supabase.com), sign up (free), create a **New Project** (any name/region; set any database password — you won't need it again).
2. Wait ~1 minute for the project to provision.

### 2. Create the tables

In the Supabase dashboard: **SQL Editor** → **New query** → paste all of this → **Run**:

```sql
create table categories (
  id uuid primary key default gen_random_uuid(),
  name text not null,
  color text not null default '#2563eb',
  created_at timestamptz default now()
);

create table tasks (
  id uuid primary key default gen_random_uuid(),
  title text not null,
  notes text,
  category_id uuid references categories(id) on delete set null,
  priority text not null default 'normal',
  due_date date,
  done boolean not null default false,
  created_by text not null,
  completed_by text,
  completed_at timestamptz,
  created_at timestamptz default now()
);

-- open access for anyone with the anon key (trusted team board)
alter table categories enable row level security;
alter table tasks enable row level security;
create policy "all categories" on categories for all using (true) with check (true);
create policy "all tasks" on tasks for all using (true) with check (true);

-- live sync
alter publication supabase_realtime add table tasks;
alter publication supabase_realtime add table categories;
```

### 3. Connect the app

1. In Supabase: **Settings → API**. Copy the **Project URL** and the **anon public** key.
2. Open `index.html`, find the `CONFIG` block near the bottom, paste both values:

```js
const CONFIG = {
  SUPABASE_URL: "https://YOURPROJECT.supabase.co",
  SUPABASE_ANON_KEY: "eyJhbGciOi...",
};
```

### 4. Publish

Commit `index.html` to the shared GitHub repo. If the repo has GitHub Pages enabled, the board is live at your Pages URL (e.g. `https://USERNAME.github.io/REPO/task-board/`). Share that link with the team.

## Notes

- The anon key is safe to commit — it's designed to be public. Anyone with the link can read/write the board, which is the point of a trusted team board. Don't use it for sensitive data.
- Supabase free tier: 500MB database, unlimited API requests — far more than a task board needs.
- To reset a user's name, click **Switch** in the top right.
