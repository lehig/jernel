# Jernal - Creative Writing & Journaling App

Jernal is a premium, glassmorphic creative writing and journaling application. It provides daily, deterministically generated prompts for users to practice their writing skills. Users can write completely anonymously, copy their work, or create an account to save their responses privately or publish them to a public community feed.

---

## ✨ Key Features

1. **Deterministic Daily Prompts**:
   - Offers two prompts every day: **Fiction** and **Personal**.
   - Uses a date-based seed calculation to cycle through prompts deterministically, ensuring all users get the same prompt on a given calendar day.
2. **Client-Side Supabase Integration**:
   - **Authentication**: Fully client-side authentication (Sign In & Sign Up) via a global glassmorphic `AuthModal`.
   - **State Persistence**: Textarea drafts are automatically backed up in `sessionStorage` so if a user signs in mid-writing, their work is never lost.
   - **Profile Dashboard**: Authenticated users can view their writing stats (total written, fiction count, personal count) and manage their entries.
   - **Community Feed**: A public page displaying chronological public writing entries shared by Jernal writers.
3. **Interactive & Aesthetic Visuals**:
   - Dark modern aesthetic with glassmorphic containers.
   - Floating particle dots and glowing blur orbs.
   - Cyberpunk-themed custom inputs, buttons, and custom category-colored checkmarks.
   - Magnetic hover micro-animations on primary call-to-action buttons.

---

## 📁 Project Structure

Here is how the codebase is organized:

```text
/
├── src/
│   ├── assets/          # Static assets (images, logos, SVGs)
│   ├── components/
│   │   ├── AuthModal.astro   # Global modal handling sign-in & sign-up
│   │   ├── Landing.astro     # Landing page details and description
│   │   ├── Navbar.astro      # Navigation with dynamic auth states
│   │   └── Prompts.astro     # Selection screen for Fiction / Personal
│   ├── data/
│   │   └── prompts.js        # Hardcoded bank of Fiction/Personal topics
│   ├── layouts/
│   │   └── Layout.astro      # Main wrapper with bg animations & global CSS
│   ├── lib/
│   │   └── supabaseClient.js # Supabase JS client configuration
│   ├── pages/
│   │   ├── feed.astro        # Community timeline of public entries
│   │   ├── index.astro       # Landing entry page
│   │   ├── profile.astro     # User dashboard for stats & entry management
│   │   └── prompt/
│   │       ├── index.astro   # Category selector routing
│   │       └── [category].astro # Writing prompt editor & submission
│   └── styles/
│       └── global.css        # Core design tokens, CSS variables, & styles
├── .env                 # Environment secrets (Supabase credentials)
├── astro.config.mjs     # Astro & Tailwind compilation config
└── package.json         # Project script & package dependencies
```

---

## 🛠️ How App Components Function

### 1. Deterministic Prompt Engine
Inside `src/pages/prompt/[category].astro`, the daily prompt is selected using the client's current date:
```javascript
const today = new Date();
const dateSeed = today.getFullYear() * 366 + today.getMonth() * 31 + today.getDate();
const promptIndex = dateSeed % categoryPrompts.length;
const activePrompt = categoryPrompts[promptIndex];
```
This calculation generates a stable number for any given calendar date, fetching the corresponding prompt index from [prompts.js](file:///c:/Users/lehig/Documents/git-jernel/src/data/prompts.js).

### 2. Session Draft Preservation
To prevent users from losing their writing when prompted to sign in, the editor listens to textarea changes and writes to `sessionStorage` under `jernel_draft_[category]`.
- Upon successful login, the `AuthModal` fires a callback that reads the draft back into the editor and immediately performs the save operation.
- Once saved, the draft is cleared from `sessionStorage`.

### 3. Dynamic Authentication Updates
The [Navbar](file:///c:/Users/lehig/Documents/git-jernel/src/components/Navbar.astro) handles auth state changes reactively by listening to Supabase's auth handler:
```javascript
supabase.auth.onAuthStateChange((event, session) => {
    if (session) {
        // Display user controls, My Profile, Sign Out
    } else {
        // Display Sign In, Start Writing
    }
});
```
This allows the UI to adapt instantly when signing in/out without forcing full page reloads.

---

## 🚀 Setup & Installation

### 1. Install Dependencies
Run this in the root directory:
```bash
npm install
```

### 2. Create the Database in Supabase
Run this SQL script in the **SQL Editor** of your Supabase Dashboard to create the `responses` table with appropriate Row Level Security (RLS) policies:

```sql
-- Create responses table
create table public.responses (
  id uuid default gen_random_uuid() primary key,
  user_id uuid references auth.users(id) on delete cascade not null default auth.uid(),
  category text not null,
  prompt_text text not null,
  response_text text not null,
  is_public boolean not null default false,
  writer_name text default 'Anonymous',
  created_at timestamp with time zone default timezone('utc'::text, now()) not null
);

-- Enable Row Level Security (RLS)
alter table public.responses enable row level security;

-- Policies for RLS
create policy "Allow public read-access for public responses"
  on public.responses for select
  using (is_public = true);

create policy "Allow read-access for own responses"
  on public.responses for select
  using (auth.uid() = user_id);

create policy "Allow insert for authenticated users on their own responses"
  on public.responses for insert
  with check (auth.uid() = user_id);

create policy "Allow update for users on their own responses"
  on public.responses for update
  using (auth.uid() = user_id)
  with check (auth.uid() = user_id);

create policy "Allow delete for users on their own responses"
  on public.responses for delete
  using (auth.uid() = user_id);
```

### 3. Setup Environment Secrets
Create a `.env` file in the root of the project:
```env
PUBLIC_SUPABASE_URL=https://your-supabase-id.supabase.co
PUBLIC_SUPABASE_ANON_KEY=your-supabase-public-anon-key
```

### 4. Running Locally
Run the Vite development server:
```bash
npm run dev
```
Open your browser at [http://localhost:4321/](http://localhost:4321/).

### 5. Production Compilation
Verify code types and compile the static bundle:
```bash
npm run build
```
This generates optimized HTML, CSS, and JS inside the `/dist` directory, ready to be deployed to static hosting providers like Netlify, Vercel, or GitHub Pages.
