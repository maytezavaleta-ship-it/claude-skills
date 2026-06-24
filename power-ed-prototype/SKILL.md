---
name: power-ed-prototype
description: Use this skill when the user wants to build a live interactive prototype of the Power Education AI assistant, recreate its look and feel, or add new feature concepts to test with customers. Triggers on phrases like "build the prototype", "add a feature to the demo", "test this concept in the prototype", "update the Power Ed site", or "add [feature] to the demo".
---

# Power Education Prototype Builder

Creates and maintains a standalone live HTML prototype of the **Power Education** AI assistant for professor Marcus Carter. The prototype faithfully recreates the look, feel, and interaction pattern of the real product so new feature concepts can be tested with customers without touching production.

---

## The core UX pattern (never break this)

Every interaction in Power Education follows this exact loop:

1. **Prompt** — professor types a question or selects a chip suggestion
2. **AI answer** — natural language reply with inline citation buttons [1][2][3]
3. **Action card** — titled card showing:
   - Card title + learner count badge
   - Learner rows: photo + name + "View profile" button
   - 2–3 action buttons, one marked **RECOMMENDED**
4. **Footer** — "Power Education Agent · [time] · Is this helpful?"

New features must fit into this loop — they extend or enhance it, never replace it.

---

## Visual design reference

| Element | Style |
|---|---|
| Background | Soft lavender-to-white gradient (`#f0eef8` → `#ffffff`) |
| Primary text | Deep navy `#1a1f5e` |
| Logo | ⚡ Power Education (navy bold) |
| Search bar | White pill, blue glow border, `+` icon left, search icon right |
| Chip buttons | Light grey pill, icon + label, hover darkens |
| AI bubble | Left-aligned, white rounded card, AI avatar top-left |
| User bubble | Right-aligned, grey bubble, user avatar top-right |
| Action card | White card, rounded, `Ready for Next Module` style heading, learner rows with circular photos |
| Action buttons | White cards with left icon, `RECOMMENDED` yellow badge on top pick |
| Sidebar | Left side, `⊞` grid icon (navigation hint) |
| Header | Logo left, user avatar + Log Out right |
| Font | System sans-serif, weights 400/600/700 |

**Learner data (always use these names for consistency):**
- Sophia Patel (Student · Advanced Biology)
- Noah Kim (Student · Advanced Biology)  
- Alex Thompson (Student · Advanced Biology)
- Marcus Carter (Professor — the logged-in user)

---

## Step 1 — Understand what to build or add

If the user says "build the prototype" with no other context, build the **base prototype** (see Step 2).

If the user says "add [feature X] to the prototype", first confirm:
- Where in the UX loop does this feature appear? (new chip, new card type, new action button, new page)
- What does the action card look like for this feature?
- Is there a new page/view (e.g. student profile, module assignment modal)?

Then proceed to implement it.

---

## Step 2 — Build or update the prototype

The prototype is a **single self-contained HTML file** at:
```
~/Desktop/power-ed-prototype/index.html
```

Create the directory if needed, then write the file.

### Base prototype spec

The base prototype must include all of these states as clickable interactions:

**Home state (default)**
- Header: ⚡ Power Education logo, user avatar, Log Out button
- Left sidebar with ⊞ grid icon
- Welcome heading: "Welcome back, Professor Carter."
- Subtitle: "Your Intro to Biology course shows strong engagement this week — 12 papers submitted since last login."
- Search bar: "Ask me anything" placeholder, `+` button left, search icon right
- Four chip buttons: Design Learning, Take Action, Understand Learners, Recommended for you
- Clicking "Recommended for you" opens a dropdown with 4 options

**Chat state — "Find learners ready for next module"**
Triggered by: clicking "How do I identify learners ready for the next module?" from the Recommended dropdown OR typing that query.

Response:
- User bubble: "How do I identify learners ready for the next module?" + timestamp
- AI bubble with avatar containing:
  - Bold line: "Sophia Patel, Noah Kim and Alex Thompson are ready for the next module."
  - Paragraph with citations: "To determine this, we looked at [how to identify students ready for the next module]¹ by reviewing several key indicators together. Their [assignment completion]² rates show consistent submission of work at or above the required threshold... Additionally, their [engagement metrics]³ reflect active participation..."
  - **Action card:**
    - Title: "Ready for Next Module" + "3 LEARNERS" badge
    - Learner rows: Sophia Patel, Noah Kim, Alex Thompson each with circular photo placeholder + "View profile" button
    - Action buttons: "Assign Next Module" (RECOMMENDED, "Assign new content to qualified learners"), "Provide Targeted Support" ("Send personalised help to struggling students"), "Send Encouragement Message" ("Motivate learners with a custom message")
  - Footer: "Power Education Agent · [time] · Is this helpful?"

**Chat state — "Sophia's latest score"**
Triggered by: typing "What is Sophia's latest score?" or similar.

Response:
- User bubble with the question
- AI bubble with avatar containing:
  - Bold line: "Sophia's latest score is 92/100 on the Cellular Biology assignment, submitted 2 days ago."
  - Supporting text citing assignment data
  - **Action card — Student Snapshot:**
    - Title: "Sophia Patel" + "Advanced Biology" badge
    - Score row: 92/100 — Cellular Biology (latest)
    - Trend: "↑ Up from 84 last assignment"
    - **"See full profile"** button (RECOMMENDED)
    - Secondary: "Send Feedback" button
  - Footer

**Student profile view**
Triggered by: clicking "See full profile" from Sophia's card OR "View profile" from the learner list.

Full-page view (replaces chat, back arrow to return):
- Student header: circular photo, "Sophia Patel", "Student · Advanced Biology · Intro to Biology"
- Stats row: Current Score 92, Assignments Completed 8/10, Engagement High, Status Ready
- Recent activity list: last 3 assignments with scores
- Action buttons: "Assign Next Module", "Send Message", "Add Note"

---

## Step 3 — Adding new features

When the user says "add [concept] to the prototype", insert it as a new interaction following this template:

### New feature template

1. **Trigger** — how does the professor invoke it? (new chip option, new prompt, new button on a card)
2. **AI answer** — what does the natural language response say?
3. **Action card** — what's the card title, what data does it show, what are the action buttons?
4. **Secondary view (optional)** — does clicking an action open a new page/modal?

Implement the feature in the existing `index.html` by:
- Adding JS state for the new interaction
- Adding the HTML template for the new card type
- Wiring the trigger to the new state
- Keeping all existing flows working

---

## Step 4 — Deploy to GitHub Pages

After writing or updating `index.html`, push it live with:

```bash
cd ~/Desktop/power-ed-prototype && git add index.html && git commit -m "Update prototype" && git push
```

GitHub Pages auto-deploys from the `main` branch — the live site updates within ~1 minute of a push.

| | |
|---|---|
| **Repo** | https://github.com/maytezavaleta-ship-it/power-ed-prototype |
| **Live URL** | https://maytezavaleta-ship-it.github.io/power-ed-prototype/ |
| **Local file** | `~/Desktop/power-ed-prototype/index.html` |

If `git push` asks for credentials, run `~/.local/bin/gh auth login` first.

---

## Step 5 — Output

After deploying:

1. Confirm the push succeeded
2. Share the live URL: `https://maytezavaleta-ship-it.github.io/power-ed-prototype/`
3. Note: "The site will update within ~1 minute."
4. List all clickable interactions available in this version
5. If new features were added, describe exactly what was added and how to trigger it

---

## Notes

- All data is hardcoded/mocked — no real API calls
- Keep it self-contained: no external dependencies (no CDN fonts, no npm). Inline all CSS and JS.
- Timestamps should be realistic (e.g. "12:54 PM") — hardcoded is fine
- Learner avatars: use colored circle placeholders with initials (SP, NK, AT) — no image files needed
- The prototype file should open directly in any browser with no server needed
