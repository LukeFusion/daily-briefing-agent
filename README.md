# Daily Briefing Agent

A fully automated morning briefing system that delivers a styled HTML email to my inbox every day at 7 AM PT. Built using **Claude Cowork** (scheduled tasks + Gmail connector) and a lightweight **Google Apps Script** relay for email delivery.

This replaced an earlier Notion-based daily review agent (see [notion-daily-review-agent](https://github.com/LukeFusion/notion-daily-review-agent)) with a simpler, zero-infrastructure approach that runs entirely on Claude's scheduling system and Google Workspace.

## What It Does

Every morning, the system:

1. Pulls today's calendar events from Google Calendar (all-day and timed, across multiple calendars)
2. Fetches the local weather forecast for La Mesa, CA
3. Compiles a 7-day "Week Ahead" view with smart tagging (busy days, holidays, travel)
4. Composes a polished HTML email with a consistent design system
5. Creates a Gmail draft with a `[CoworkBrief]` subject prefix
6. A Google Apps Script trigger detects the draft and auto-sends it

The email lands in my inbox by ~7:00 AM Pacific, ready to scan on my phone before I start the day.

## Architecture

```
┌─────────────────────────────────────────────────────────┐
│                  Claude Cowork Platform                  │
│                                                         │
│  ┌───────────────────┐    ┌──────────────────────────┐  │
│  │  Scheduled Task   │    │    Gmail Connector        │  │
│  │  (daily @ 6:58am) │───>│    (create_draft)         │  │
│  │                   │    │                           │  │
│  │  • Google Calendar│    │  Creates draft with       │  │
│  │  • Weather search │    │  [CoworkBrief] prefix     │  │
│  │  • HTML compose   │    └───────────┬──────────────┘  │
│  └───────────────────┘                │                  │
└───────────────────────────────────────┼──────────────────┘
                                        │
                                        ▼
                              ┌──────────────────┐
                              │   Gmail Drafts   │
                              │   (holding zone)  │
                              └────────┬─────────┘
                                       │
                                       ▼
┌──────────────────────────────────────────────────────────┐
│              Google Apps Script Relay                     │
│                                                          │
│  Trigger: every 5 min                                    │
│  Action:  find drafts with [CoworkBrief] prefix          │
│           → send email                                   │
│           → strip prefix from subject                    │
│           → delete draft                                 │
│                                                          │
└──────────────────────────────────────────────────────────┘
                                       │
                                       ▼
                              ┌──────────────────┐
                              │    📬 Inbox      │
                              │    ~7:00 AM PT    │
                              └──────────────────┘
```

### Why This Architecture?

Claude Cowork's Gmail connector supports reading emails and creating drafts, but **cannot send emails directly** (as of April 2026). The Apps Script relay bridges that one gap with a ~30-line script running on Google's servers. No external infrastructure, no API keys, no cron servers.

## Email Design

The briefing uses a consistent HTML design system optimized for mobile email clients:

- **Green palette** (`#2d6a4f` primary, `#52b788` accent, `#e8f5ee` highlights) with navy accents (`#1b4965`)
- **Header/footer sandwich** with a gradient accent bar for a "fixed report" feel
- **Weather card** with fallback messaging if data is unavailable
- **Today's Schedule** with all-day vs. timed event separation, colored left-border cards, and calendar source pills
- **Week Ahead** section with "Busy Day" badges (3+ events), holiday/birthday callouts, and location details
- **Controlled emoji usage** — weather icons, celebrations, travel markers only
- Inline CSS, table-based layout for email client compatibility

![Morning Brief Screenshot](assets/morning-brief-screenshot.png)

## Components

| Component | Location | Purpose |
|-----------|----------|---------|
| Scheduled task prompt | Claude Cowork (sidebar → Scheduled) | Orchestrates data gathering and email composition |
| Apps Script relay | Google Apps Script ([script.google.com](https://script.google.com)) | Auto-sends drafts with `[CoworkBrief]` prefix |
| HTML template spec | Embedded in scheduled task prompt | Design system and layout rules |

## Setup

### Prerequisites

- [Claude Desktop](https://claude.ai) with Cowork mode enabled
- Gmail connector authorized on your account
- Google Calendar connector authorized
- A Google account for Apps Script

### 1. Apps Script Relay (~5 min)

1. Go to [script.google.com](https://script.google.com) → **New Project**
2. Paste the contents of [`apps-script/autosend-relay.gs`](apps-script/autosend-relay.gs)
3. Save and name the project (e.g., "Cowork Briefing Relay")
4. Run `sendCoworkDrafts` once manually to authorize Gmail access
5. Add a trigger: Triggers → **+ Add Trigger** → `sendCoworkDrafts`, time-driven, every 5 minutes

Google will show an "unverified app" warning since this is a personal script. Click **Advanced → Go to [project name]** to authorize. This is normal for personal Apps Scripts.

### 2. Scheduled Task

The scheduled task is created and managed within Claude Cowork. It runs daily at ~6:58 AM PT and handles all data gathering, HTML composition, and draft creation automatically.

The task prompt contains the full design specification, data gathering steps, and HTML template rules. See the [Architecture](#architecture) section for how the pieces connect.

## Roadmap

Planned expansions to the briefing content:

- [ ] Pending to-do items (from Notion or task system)
- [ ] Daily quote or joke
- [ ] Goal progress tracking
- [ ] Reply-back workflow — reply to the briefing email with instructions, and a second scheduled task reads your reply and takes action (create events, draft follow-ups, update Notion, etc.)

## Background

This project evolved from an earlier [Notion-based daily review agent](https://github.com/LukeFusion/notion-daily-review-agent) built with OpenAI Codex. That version used Notion as both the data store and delivery surface. This version simplifies the architecture by using Claude Cowork's native scheduling and Gmail integration, with email as the primary interface — making it accessible from any device without opening a specific app.

The key insight: email is a universal inbox. By routing the briefing through Gmail, it becomes scannable on a phone lock screen, and the reply-back pattern opens up a conversational interface to the agent without needing a separate app.

## Author

**Luke Squire** — Product leader exploring agentic AI workflows for personal productivity and professional systems.

- [GitHub](https://github.com/LukeFusion)
- [LinkedIn](https://linkedin.com/in/lukesquire)
