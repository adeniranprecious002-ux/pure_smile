# PureSmile Dental Hospital — Patient Automation System

> End-to-end patient communication automation built with n8n, Google Sheets, OpenAI, and Gmail.
> Built as a portfolio project based on a real $1,000 Upwork job brief.

![n8n](https://img.shields.io/badge/n8n-Cloud-teal?style=flat-square)
![OpenAI](https://img.shields.io/badge/OpenAI-GPT--4o--mini-412991?style=flat-square)
![Gmail](https://img.shields.io/badge/Gmail-API-red?style=flat-square)
![Google Sheets](https://img.shields.io/badge/Google_Sheets-CRM-34A853?style=flat-square)
![Status](https://img.shields.io/badge/Status-Production_Ready-brightgreen?style=flat-square)

---

## What This Is

A complete automation system for small dental and medical clinics. It handles the full patient lifecycle automatically — from the moment a new enquiry arrives to long-term reactivation of inactive patients — with zero manual intervention from clinic staff.

**Live demo:** Open `dental_hospital_website.html` in any browser.

---

## What's Included

```
puresmile-patient-automation/
├── Pure_smile.json                  # n8n workflow export — all 5 workflows
├── dental_hospital_website.html     # Complete patient-facing website
├── README.md                        # This file
└── docs/
    ├── email-templates/             # All 10 HTML email template sources
    └── screenshots/                 # Workflow canvas and email screenshots
```

---

## The 5 Workflows

| Workflow | Trigger | What It Does |
|---|---|---|
| **Module 1** — New Patient Follow-Up | Webhook (POST) | Saves lead → welcome email → staff notification → Day 3 nudge |
| **Module 2** — Appointment Reminders | Schedule (every 15 min) | Reads Sheets → filter 24hr appts → remind → 2hr nudge → cancel |
| **Module 3** — Google Form Booking | Form submission | Update Sheets → send booking confirmation email |
| **Module 4** — Patient Reactivation | Schedule (Monday 9am) | Filter 6-month inactive → OpenAI email → 5-day follow-up → dormant |
| **Module 5** — Error Handler | Error Trigger | Catch any failure → send alert email with workflow + node + error |

---

## Tech Stack

| Tool | Role |
|---|---|
| [n8n Cloud](https://n8n.io) | Workflow automation engine |
| [Google Sheets](https://sheets.google.com) | Patient CRM database |
| [Google Forms](https://forms.google.com) | Patient appointment booking |
| [OpenAI GPT-4o-mini](https://platform.openai.com) | AI-personalised reactivation emails |
| [Gmail API](https://developers.google.com/gmail) | All email delivery |
| HTML / CSS / JS | Patient-facing website |

---

## Quick Start

### 1. Set up Google Sheets

Create a new Google Sheet called `PureSmile` with a sheet tab named `Client DB`.

Add these column headers in row 1 exactly as shown (spelling and spacing matter):

```
First Name | Last Name | Email address | Number | Message | Service |
Preffered Date | Preffered Time | Submitted At | Status | Booking Confirmed | Last Notified At
```

### 2. Import the n8n workflow

1. Open your n8n instance
2. Click **Settings** → **Import Workflow**
3. Upload `Pure_smile.json`
4. All 5 workflows will import together

### 3. Connect your credentials

In n8n, go to **Credentials** and add:

| Credential | Type | Used by |
|---|---|---|
| Google Sheets | OAuth2 | All Sheets nodes |
| Gmail | OAuth2 | All Gmail nodes |
| OpenAI | API Key | Reactivation workflow |

### 4. Update the webhook URL in the website

Open `dental_hospital_website.html` in a text editor. Find the `handleSubmit` function near the bottom and replace the webhook URL:

```javascript
await fetch('YOUR_N8N_WEBHOOK_URL_HERE', {
  method: 'POST',
  headers: { 'Content-Type': 'application/json' },
  body: JSON.stringify(formData)
});
```

Your n8n webhook URL is found in the Webhook node of Module 1.

### 5. Update clinic details in email templates

Search and replace throughout the workflow Gmail nodes:

- `PureSmile Dental Hospital` → your clinic name
- `14 Admiralty Way, Lekki Phase 1, Lagos` → your address
- `+234 800 123 4567` → your phone number
- `YOUR_GOOGLE_FORM_LINK` → your Google Form booking link

### 6. Publish all workflows

In n8n, open each workflow and click **Publish**. The workflows won't run until published.

---

## Google Sheets Setup — Field Reference

| Column | Format | Notes |
|---|---|---|
| First Name | Text | Used in email personalisation |
| Last Name | Text | |
| Email address | Text | **Primary key** — all row updates match by this field |
| Number | Text | Store as text to preserve leading zeros |
| Message | Text | Notes from website form |
| Service | Text | Must match doctor map in Code node exactly |
| Preffered Date | YYYY-MM-DD | Date format is critical for reminder filter |
| Preffered Time | Text | e.g. "10:30 AM" |
| Submitted At | ISO timestamp | Auto-filled by workflow |
| Status | Text | new-lead / booked / reactivation-sent / dormant / unsubscribed |
| Booking Confirmed | Yes / No | Updated by appointment workflow |
| Last Notified At | YYYY-MM-DD | **Must be populated** for reactivation filter to work |

> **Important:** Backfill `Last Notified At` for existing patients with a date older than 6 months (e.g. `2025-10-01`) before running the reactivation workflow for the first time. Rows with an empty `Last Notified At` are also caught by the OR condition in the filter.

---

## Common Issues & Fixes

**Filter returns no output**
→ `Last Notified At` column is blank. Add a date (YYYY-MM-DD) to existing rows.

**Reactivation crashes on row 2 with "item index" error**
→ You're referencing the Date & Time node without `.first()`. Change:
`{{ $node["Date & Time"].json.cutoffDate }}` to `{{ $('Date & Time').first().json.cutoffDate }}`

**Form hangs after submission**
→ Add a **Respond to Webhook** node in Module 1 and connect it to the Webhook node as a parallel branch. Set it to respond with `{ "status": "received" }`.

**Reminders not sending / wrong patients getting emails**
→ Check that the initial `status` in Edit Fields is set to `new-lead` (lowercase). If set to `Booked`, the Switch node routes all patients to the confirmed branch and skips reminders.

**Wait nodes firing immediately**
→ Open each Wait node and check that Amount and Unit are set. Empty Wait nodes default to 0 and fire instantly.

**Gmail node not sending**
→ OAuth2 token may have expired. Go to n8n Credentials → Gmail → re-authenticate.

**Staff notification shows no name**
→ The field reference should be `$('Edit Fields').item.json.first_name` and `$('Edit Fields').item.json.last_name` with a space between them.

---

## Customising the Doctor Map

The Code node in Module 2 maps service type to doctor name and bio. To update doctors, open the Code node and edit the `docMap` object:

```javascript
const docMap = {
  "Orthodontics": {
    name: "Dr. Amara Nwosu",
    bio: "MSc London, specialising in Invisalign."
  },
  "Dental implants": {
    name: "Dr. James Okafor",
    bio: "Fellowship in Vienna, over 3,000 procedures."
  },
  "Cosmetic dentistry": {
    name: "Dr. Chisom Eze",
    bio: "Award-winning smile designer."
  },
  "Paediatric dentistry": {
    name: "Dr. Fatima Bello",
    bio: "UCL Diploma, expert in children's care."
  },
  "General dentistry": {
    name: "Dr. [Name]",
    bio: "[Bio here]"
  }
};
```

The service name must exactly match what appears in the `Service` column of Google Sheets.

---

## Deploying the Website

The website is a single HTML file with no dependencies. To deploy:

**Option A — Netlify (recommended, free)**

1. Go to [netlify.com](https://netlify.com)
2. Drag and drop `dental_hospital_website.html` onto the deploy area
3. Done — live in 30 seconds

**Option B — GitHub Pages**

1. Create a repo, rename the file to `index.html`
2. Go to Settings → Pages → Deploy from main branch

**Option C — Any web host**
Upload the file via cPanel or FTP. No server-side requirements.

---

## Connecting the Error Handler

After importing, link the Error Handler to all other workflows:

1. Open **Module 1 — New Patient Follow-Up**
2. Click the three-dot menu → **Settings**
3. Under **Error Workflow**, select **PureSmile — Error Handler**
4. Repeat for Modules 2, 3, and 4

The Error Handler must be **Published** before it can be selected.

---

## Testing

Before going live, test each workflow in order:

```
1. Submit the website form → check Sheets row added + welcome email received
2. Add a test row with tomorrow's date → run Module 2 manually → check reminder email
3. Submit Google Form → check Status updated to Booked + confirmation email received
4. Set Last Notified At to a date older than 6 months → run Module 4 manually → check reactivation email
5. Delete the To field in any Gmail node → run the workflow → check error alert email received
```

---

## Project Background

This system was built as a portfolio project based on a real Upwork job posting for an AI Automation Specialist to build patient communication workflows for a dental clinic. Budget: $500–$1,500.

Full write-up on Medium: [[Medium](https://medium.com/@adeniranprecious002/i-found-a-1-000-upwork-job-post-and-built-the-entire-system-before-applying-7f50c7d1d4df)]
LinkedIn build series: [[LinkedIn] (https://www.linkedin.com/in/precious-adeniran-842b58294)]

---

## Licence

MIT — free to use, adapt, and deploy for any project.

---

## Contact

Built by [Adeniran Precious Adebayo]

- LinkedIn: [[LinkedIn](https://www.linkedin.com/in/precious-adeniran-842b58294)]
- Upwork: [[Upwork](https://upwork.com/freelancers/~01721e743c53bddd4e)]
- Email: [[Mail](adeniranprecious002@gmail.com|)]

If you found this useful, a ⭐ on the repo goes a long way.
