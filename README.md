# Reflective Time-Block Planner

A personal calendar and planner built around three ideas that existing tools handle poorly: tasks that span **multiple time blocks**, **subtasks assigned to specific blocks**, and a **reflective view** that records how the day actually went against how it was planned. The app is designed to run inside Notion so tasks and materials live in one place.

## The problem

In Google Calendar or Notion, a task worked on in separate sessions — for example coding before lunch and again after — has to be created as several independent entries and checked off several times. There is no single task that owns several time blocks. Existing time-blocking tools each solve part of this (ClickUp allows one task in multiple slots; Sunsama and Toggl offer reflection or planned-vs-actual tracking), but none combine all three features, and none embed cleanly into a Notion workspace.

## Features

- **One task, many time blocks.** A task is created once and checked off once, even when its work is split across the day.
- **Subtasks per block.** Subtasks belong to a parent task but can be assigned to specific blocks, so a subtask can be scheduled into the morning block while another is scheduled into the afternoon block of the same task.
- **Reflective schedule.** Alongside the planned schedule, an "actual" schedule records what really happened — planning a task for 09:00–10:00 but working on it 09:45–12:00 produces an actual block that reflects reality, for review at the end of the day.
- **Notion integration.** The calendar is embedded into a Notion page for viewing and editing, and tasks sync with a Notion database in both directions.

## Tech stack

The stack is chosen to run at zero cost for single-user personal use, with a clear paid upgrade path if the project ever grows.

| Layer | Choice | Role |
|-------|--------|------|
| Frontend | Next.js (React) + TypeScript | The interactive timeline, blocks, and checkboxes |
| Styling | Tailwind CSS + shadcn/ui | Prebuilt, consistent UI components |
| Database | Supabase (hosted Postgres) | Stores tasks, blocks, subtasks, and planned/actual records |
| Backend / API | Supabase (auto-generated) | Serves data to the frontend without a separate server |
| Auth | Supabase Auth | Per-user access |
| Hosting | Vercel (hobby tier) | Deploys the app to a public URL, auto-deploys on push |
| Notion | Notion API (internal integration) | Two-way task sync with a Notion database |

Notes on the free tier: Supabase pauses an inactive project after about seven days (a non-issue for daily use), and Notion iframe embeds are unreliable in the Notion mobile app, so a plain browser link is kept as a mobile fallback. The Notion API is rate-limited, so the app's own database is the source of truth and Notion is synced to it rather than used as the live backend.

## Data model

Five tables carry the core structure:

- **`tasks`** — the parent unit that is checked off once. Fields: `id`, `title`, `done`, `created_at`, `notion_page_id`.
- **`time_blocks`** — many per task. Fields: `id`, `task_id`, `date`, `start_time`, `end_time`, `done`, `kind` (`planned` or `actual`), `reflects_block_id` (an actual block may point to the planned block it corresponds to).
- **`subtasks`** — `id`, `task_id`, `title`, `done`.
- **`subtask_block_assignments`** — join table linking a `subtask_id` to a `block_id`, which is what allows a subtask to be assigned to one specific block of its parent task.

The `kind` field on `time_blocks` is what allows the planned and actual schedules to coexist in one table and be overlaid in the reflective view.

## Getting started

Requirements: Node.js (LTS), Git, and a code editor.

### 1. Create the project

```bash
npx create-next-app@latest planner   # choose TypeScript and Tailwind CSS
cd planner
git init
git add .
git commit -m "initial commit"
npx shadcn@latest init
```

Create an empty repository on GitHub and push using the commands GitHub provides for an existing repo.

### 2. Configure Supabase

1. Create a project at supabase.com and save the generated database password.
2. Under **Project Settings → API**, copy the **Project URL** and **anon public key**.
3. Add them to `.env.local` as `NEXT_PUBLIC_SUPABASE_URL` and `NEXT_PUBLIC_SUPABASE_ANON_KEY` (this file is git-ignored — secrets stay out of the repo).
4. Install the client:

```bash
npm install @supabase/supabase-js
```

5. Create the five tables above in the Supabase Table Editor, enable Row Level Security, and add a policy restricting each user to their own rows.

### 3. Deploy

1. Create a Vercel account and import the GitHub repository.
2. Add the Supabase environment variables in the Vercel project settings.
3. Deploy. Each push to the repository redeploys automatically.

### 4. Connect Notion (after the app works standalone)

1. Create an internal integration at notion.so/my-integrations and copy its token.
2. In Notion, open the target database and connect it to the integration under **⋯ → Connections**.
3. Store the token as a Vercel environment variable — never in client-side code or the repository.
4. Implement one direction first: creating a task in the app creates a matching row in the Notion database, storing the returned page id in `tasks.notion_page_id`. Add the reverse direction once that is stable.
5. Embed the app in a Notion page with `/embed` and the deployed URL, keeping a plain link alongside it for mobile.

## Roadmap

1. Static day view with placeholder blocks to establish the timeline UI.
2. Connect Supabase; create and read real tasks and blocks.
3. Add subtasks and per-block assignment.
4. Add the planned-vs-actual reflective view.
5. Add authentication.
6. Add the Notion embed, then Notion sync.

Development starts from the day timeline — a vertical list of hours with blocks drawn in, a control to create a task, and a checkbox on each block. The remaining features build outward from that view.
