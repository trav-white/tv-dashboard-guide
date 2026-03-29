# Building TV Dashboards with Claude — A Practical Guide

**From**: Trav White, Neighbourhood (NBH)
**For**: Daniel (and anyone else who wants live dashboards on their office TVs)

---

## What I Built

I run Neighbourhood (NBH) — we're a specialist HubSpot consultancy and Diamond Partner based in Brisbane. We help businesses (including allied health and healthcare providers) get their CRM, operations, and reporting dialled in.

Internally, we built our own operating system called NBHOS — a platform that runs our entire business. One piece of that is a set of TV dashboards that display live business data on screens around the office: sales pipeline, revenue, team capacity, marketing performance, client health scores, and more.

The whole thing was built using Claude (Anthropic's AI). I'm not a developer — I described what I wanted, Claude wrote the code, and I iterated on it until it worked. The result is 14 dashboard screens that auto-rotate on TVs throughout the office, refreshing every 2-5 minutes with live data.

### What it looks like in practice

- Dark-themed, large-font displays optimised for reading from across a room
- KPI cards with animations (numbers fade and slide in on refresh)
- Auto-rotating slides — each dashboard cycles through 2-4 views every 12 seconds
- Charts, leaderboards, progress bars, client lists
- A digital signage screen that rotates images, videos, motivational quotes, and dashboard widgets
- Everything pulls from a database (no live API calls from the screen — fast and reliable)

### The tech stack

| Layer | What I used | What it does |
|-------|------------|--------------|
| **Framework** | Next.js (React) | The web app that renders the dashboards |
| **Database** | PostgreSQL | Stores all the data the dashboards display |
| **Animations** | Framer Motion | Smooth fade/slide/stagger animations on the cards |
| **Styling** | CSS variables + inline styles | Dark theme, responsive sizing, TV-optimised typography |
| **Hosting** | DigitalOcean (server) | Runs the app and database |
| **TV Hardware** | Amazon Fire TV Stick | Cheap, reliable, plugs into any TV with HDMI |
| **Kiosk Browser** | Fully Kiosk Browser (Android) | Locks the Fire Stick into showing one URL, auto-restarts on crash |

---

## What You Need

### Hardware

| Item | Cost (approx) | Notes |
|------|---------------|-------|
| **Amazon Fire TV Stick** (4K or Lite) | $50-80 AUD | One per TV. The Lite works fine. |
| **TV with HDMI input** | You probably already have one | Any size works. Bigger = more impressive. |
| **Wi-Fi network** | Existing | The Fire Stick needs internet to load the dashboard |

The Fire TV Stick is the move here. It's cheap, it runs Android apps, and it just works. You could also use a Raspberry Pi, an old laptop, or a Chromecast — but the Fire Stick + Fully Kiosk combo is the most reliable setup I've found.

### Software & Services

| Service | Cost | Purpose |
|---------|------|---------|
| **Claude Pro or Max** | $20-200 USD/month | The AI that builds your dashboards |
| **Vercel** (recommended for you) | Free tier to start | Hosts your Next.js app on the internet |
| **GitHub** | Free | Stores your code, connects to Vercel for auto-deploy |
| **Fully Kiosk Browser** | ~$7 USD (one-time, per device) | Android app that locks the Fire Stick to your dashboard URL |
| **Database** (Vercel Postgres or Supabase) | Free tier to start | Stores the data your dashboards display |

**Total ongoing cost**: ~$20-200/month (just Claude). Everything else is free or one-time.

### Accounts to set up before you start

1. **Claude** — [claude.ai](https://claude.ai) (Pro plan minimum for extended conversations)
2. **GitHub** — [github.com](https://github.com) (free account)
3. **Vercel** — [vercel.com](https://vercel.com) (free tier, sign in with GitHub)
4. **Supabase** or **Vercel Postgres** — for your database (free tier)

---

## Before You Start: What Data Do You Want on Your TVs?

This is the most important step and it's got nothing to do with code. Sit down and think about what numbers you want your team to see every day.

**For an allied health practice, you might want things like:**

- Today's appointment count / capacity %
- Revenue this week/month vs target
- New patient bookings this week
- Cancellation/no-show rate
- Practitioner utilization (hours booked vs available)
- Outstanding invoices / accounts receivable
- Google review count and average rating
- Marketing leads (website enquiries, phone calls)
- Waitlist length by service type
- Team birthdays, announcements, motivational quotes

**Ask yourself:**
- What do I check every morning? Put that on the dashboard.
- What do I wish the team could see without asking me? Put that on the dashboard.
- What numbers drive behaviour? (e.g., if the team sees cancellation rate is high, do they follow up harder?)
- Where does this data currently live? (Practice management software, Xero, Google Sheets, CRM, etc.)

**Write down:**
1. The 5-10 metrics you care about most
2. Where each metric currently lives (which software/spreadsheet)
3. How often it needs to refresh (real-time? hourly? daily?)

You'll give this list to Claude when you start building.

---

## Step-by-Step: Building Your First Dashboard

### Phase 1: Set Up Your Development Environment

You don't need to install coding tools on your computer. Use **Claude Code** (the CLI) or **Claude.ai with the cloud environment** to write and run code directly.

**Option A — Claude.ai Cloud (easiest)**

1. Go to [claude.ai](https://claude.ai) and sign in
2. Start a new conversation
3. Use the cloud environment feature (Claude can create and edit files in a sandbox)
4. When your app is ready, push it to GitHub and deploy to Vercel

**Option B — Claude Code on your Mac (more control)**

1. Install Claude Code: open Terminal and run the installer from [claude.ai/code](https://claude.ai/code)
2. Create a project folder: `mkdir ~/tv-dashboard && cd ~/tv-dashboard`
3. Start Claude Code: `claude`
4. Claude will write files directly on your machine

Either way works. Option A requires zero setup. Option B gives you more control and is what I use.

### Phase 2: Create the Base App

Start a conversation with Claude and give it context. Here's a prompt template you can adapt:

---

**Your first prompt to Claude:**

> I want to build a TV dashboard web app for my allied health practice. It will be displayed on a TV in the office using an Amazon Fire TV Stick running Fully Kiosk Browser.
>
> **Tech stack:**
> - Next.js (App Router) with TypeScript
> - Tailwind CSS for base styling, but inline styles with CSS variables for the dashboard components
> - Framer Motion for animations
> - PostgreSQL database (I'll use Supabase/Vercel Postgres)
>
> **Display requirements:**
> - Dark theme (easy on the eyes, looks good on a TV)
> - Large fonts readable from 3-4 metres away
> - Full-screen layout, no sidebar or navigation chrome
> - Auto-refreshes data every 5 minutes without reloading the page
> - If there are multiple "slides" of data, auto-rotate between them every 12 seconds with a crossfade transition
> - Optimised for 1920x1080 resolution
>
> **My first dashboard should show:**
> [LIST YOUR METRICS HERE — e.g.:]
> - Today's appointments: booked vs capacity (as a percentage)
> - Revenue this month vs monthly target (with a progress bar)
> - New patient bookings this week
> - Cancellation rate this week
> - Top 5 practitioners by utilization this week
>
> **Data source:**
> For now, create a simple API route that returns mock/seed data so I can see the dashboard working. I'll connect real data later.
>
> Please create the full project structure, the dashboard page at /tv, and the API route at /api/tv/dashboard.

---

Claude will generate the project for you. Review what it builds, ask it to adjust anything you don't like (colours, layout, font sizes, animation speed), and iterate.

### Phase 3: Get It Running Locally (or in Cloud)

If you're using Claude Code locally:

```
npm install
npm run dev
```

Then open `http://localhost:3000/tv` in your browser. You should see your dashboard with mock data.

If you're in the cloud environment, Claude will handle this for you.

**Iterate here.** This is where you spend time getting the look and feel right. Ask Claude things like:

- "Make the KPI cards bigger, they need to be readable from further away"
- "Add a slide that shows a bar chart of appointments by practitioner"
- "The animation is too fast, slow it down"
- "Add a header with our practice name and a 'last updated' timestamp"
- "Change the accent colour to [your brand colour]"

Don't try to get everything perfect in one go. Build one slide, get it looking good, then add more.

### Phase 4: Connect Real Data

This is where it gets specific to your setup. The general pattern is:

1. **Your practice management software** (Cliniko, Nookal, Halaxy, etc.) likely has an API
2. **Create a sync job** — a scheduled task that pulls data from that API and stores it in your database
3. **Your dashboard reads from the database** — never directly from the external API

**Why the database layer?** If your practice software API goes down or is slow, your TV dashboard still works. The database always has the last-known-good data.

**Prompt to Claude:**

> I use [YOUR SOFTWARE, e.g., Cliniko] for practice management. It has a REST API.
> I want to create a cron job / scheduled function that runs every 15 minutes, pulls today's appointments, revenue, and practitioner schedules from the Cliniko API, and stores the results in my PostgreSQL database.
> Then update my /api/tv/dashboard route to read from the database instead of returning mock data.

If your data lives in **Google Sheets** (a lot of small businesses track KPIs there), that's even easier:

> My KPI data is in a Google Sheet. I want to sync that sheet to my database every 15 minutes and display it on the dashboard.

Claude will walk you through the API setup, authentication, and data sync.

### Phase 5: Deploy to the Internet

Your TV needs to load the dashboard from a URL, so you need to deploy it.

**Using Vercel (recommended — free, simple):**

1. Push your code to GitHub (Claude can help: "help me push this to a new GitHub repo")
2. Go to [vercel.com](https://vercel.com), sign in with GitHub
3. Import your repo
4. Add your environment variables (database URL, API keys) in Vercel's dashboard
5. Deploy

Vercel gives you a URL like `your-app.vercel.app`. That's what your TV will load.

**Add basic security** so random people can't see your business data:

> Add a simple token-based auth to the /tv route. It should accept a ?token= query parameter and check it against an environment variable called TV_SECRET. If the token is wrong, show a blank page.

Your TV URL becomes: `https://your-app.vercel.app/tv?token=your-secret-here`

### Phase 6: Set Up the Fire TV Stick

1. **Plug in the Fire TV Stick** and complete the basic setup (Wi-Fi, Amazon account)
2. **Install Fully Kiosk Browser**:
   - On the Fire Stick, go to Settings > My Fire TV > Developer Options > enable "Apps from Unknown Sources"
   - Search for "Downloader" in the Amazon App Store and install it
   - Open Downloader, enter the URL for Fully Kiosk Browser APK: `https://www.fully-kiosk.com/apk/`
   - Install the APK
3. **Configure Fully Kiosk Browser**:
   - Set the Start URL to your dashboard: `https://your-app.vercel.app/tv?token=your-secret-here`
   - Enable kiosk mode (locks the device to just showing the browser)
   - Enable "Launch on Boot" (so if the TV loses power and comes back, it auto-loads)
   - Set screen orientation to Landscape
   - Disable the screen saver or set it to reload the page
4. **Done.** The TV now shows your dashboard permanently.

**Fully Kiosk costs about $7 USD per device** (one-time licence). It's worth it — it handles crashes, auto-restarts the browser, and prevents the Fire Stick from going to the home screen.

---

## Adding More Dashboards

Once your first dashboard is working, you can add more screens at different routes:

- `/tv` — Main dashboard (appointments, revenue)
- `/tv/team` — Practitioner utilization and schedules
- `/tv/marketing` — Lead sources, Google reviews, website traffic
- `/tv/signage` — Rotating announcements, team birthdays, quotes

Each TV can show a different dashboard by pointing its Fully Kiosk URL to a different route. Or you can build a carousel that cycles through all of them.

**Prompt to Claude:**

> I want to add a second dashboard at /tv/team that shows practitioner utilization. It should have the same dark theme, header, and animation style as the main dashboard. Show each practitioner's name, hours booked today, hours available, and a utilization % bar.

---

## Tips From Having Done This

**On working with Claude:**

- **Be specific about the visual feel.** "Dark theme" is vague. "Dark background (#0a0a0f), warm off-white text, subtle card borders, no harsh whites" gets you what you want faster.
- **Iterate in small steps.** Build one card, get it right, then ask for more. Don't try to describe the entire dashboard in one prompt.
- **Show Claude screenshots.** If you see something you like on Dribbble or another dashboard, screenshot it and say "make it look like this."
- **Don't worry about "doing it right."** There's no wrong way to prompt. If the result isn't what you want, just tell Claude what to change.
- **Save your prompts.** When you find a prompt that produces a good result, save it. You'll reuse patterns across dashboards.

**On the hardware:**

- **Fire TV Stick Lite is fine.** You don't need the 4K version for a dashboard — it's just rendering a web page.
- **Use the query parameter for auth, not just cookies.** Fire Sticks occasionally clear cookies when they go to sleep. The `?token=` in the URL persists.
- **Expect the occasional restart.** Fully Kiosk handles this gracefully, but about once a month the Fire Stick might need a power cycle. Not a big deal.
- **Hardwire if you can.** Amazon sells a Fire TV ethernet adapter (~$25). More reliable than Wi-Fi if the TV is near your router.

**On the data:**

- **Start with mock data.** Get the dashboard looking good with fake numbers before you try to connect real APIs. This keeps the visual design separate from the data plumbing.
- **Google Sheets is a valid data source.** If you already track KPIs in a spreadsheet, just sync that to your database. No shame in it.
- **Cache everything.** Your dashboard should never call an external API directly. Always go through your database. External APIs are slow, rate-limited, and go down.

---

## What This Costs to Run (Ongoing)

| Item | Monthly Cost |
|------|-------------|
| Claude Pro (for building/maintaining) | $20-200 USD |
| Vercel hosting (free tier) | $0 |
| Supabase database (free tier) | $0 |
| Fully Kiosk Browser | $0 (one-time $7/device) |
| Fire TV Stick | $0 (one-time $50-80/device) |
| **Total ongoing** | **$20-200/month** (just Claude) |

Once it's built, you only need Claude when you want to change or add things. The dashboard itself runs for free on Vercel's free tier for small-scale use.

---

## Getting Started Today

1. Sign up for Claude Pro at [claude.ai](https://claude.ai)
2. Write down the 5-10 metrics you want on your first dashboard
3. Note where each metric lives (which software, spreadsheet, etc.)
4. Buy a Fire TV Stick and a Fully Kiosk licence
5. Start a conversation with Claude using the prompt template in Phase 2
6. Iterate until it looks good, deploy to Vercel, point the Fire Stick at it

The first dashboard will take the longest because you're learning the process. After that, each new one gets faster because you (and Claude) can reuse the patterns from the first.

---

## Questions?

Reach out to Trav — happy to show you the live setup or help you get started.

**Trav White**
Neighbourhood (NBH) — nbh.co
