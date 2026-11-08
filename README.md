# Smart Clinic AI Assistant

**DEPI — Digital Egypt Pioneers Initiative | Track: AI & Data Science**

An n8n-based automation system for a single-doctor clinic. Patients talk to a Telegram bot to ask about the clinic and book appointments, an AI agent answers free-text questions, and scheduled workflows handle reminders, the daily financial report, and post-visit feedback.

**Try the bot:** https://t.me/YOUR_BOT_USERNAME *(works only while the n8n workflow is active)*

---

## Features

| Module | What it does | Status |
|---|---|---|
| 1. Inquiries & booking | Button-driven Telegram menu for inquiries (address, fees, working hours) and booking against real Google Calendar availability. An AI agent answers free-text questions (clinic information only) | Working |
| 2. Reminders | Sends an interactive reminder 24 hours before each appointment: Confirm / Reschedule / Cancel | In progress |
| 3. Daily financial report | Computes daily revenue and case count in code (not by the LLM) and emails the doctor via Gmail | In progress |
| 4. Patient feedback | Requests feedback after the visit; an LLM classifies it as Positive, Neutral, or Negative | In progress |

**Design principle:** decisions that must be exact (availability, bookings, money) are made in code. The LLM only understands and produces language: it answers free-text questions with clinic information, and classifies feedback sentiment. It never books an appointment and never calculates a number.

## How booking works

1. The patient taps **Book** and the bot offers the next working days (Friday and Saturday are skipped).
2. After the patient picks a day, the workflow reads that day's events from Google Calendar.
3. Code generates 20-minute slots between 09:00 and 15:00 (Africa/Cairo time), removes past slots and slots that overlap existing events, and shows the rest as buttons.
4. When the patient picks a slot, the workflow **remembers it for that chat** and asks for name and age. The patient can send them as a normal message or as a reply. The remembered slot expires after 30 minutes and is cleared by `/start` or any menu button.
5. The bot validates the data (name of at least three words, age from 1 to 110) and shows a summary to confirm.
6. On confirmation, the slot is checked **again** against the calendar. If it is still free, an event is created; otherwise the patient is told the slot was just taken. This second check prevents double booking.

## Free-text questions (AI agent)

Messages that are not a button press, not a command, and not part of an active booking step go to an AI agent. It answers from clinic information only, has no booking tools, gives no medical advice, and points the patient to `/start` to book. If the AI service is unavailable, the bot falls back to the menu.

## Tech stack

- **n8n** — workflow automation engine (including the AI Agent node)
- **Telegram Bot API** — patient interface
- **Google Calendar API** — availability and appointments
- **Google Sheets API** — data store for bookings, ledger, and feedback
- **Gmail API** — daily report delivery
- **LLM API** — AI agent for free-text answers, and feedback sentiment classification

## System requirements

- An n8n instance (n8n Cloud, or self-hosted) reachable over a **public HTTPS URL**. The Telegram Trigger cannot receive messages from a plain `localhost`.
- A Telegram account, to create a bot with `@BotFather`.
- A Google account with Google Calendar (and later Sheets and Gmail).
- An API key (or a reachable model) for the LLM used by the AI agent and the feedback module.

## Installation and setup

1. **Import the workflow.** In n8n, open Workflows, choose **Import from file**, and select the workflow JSON from the `workflows/` folder.
2. **Create your Telegram bot.** In Telegram, message `@BotFather`, send `/newbot`, and copy the token it gives you.
3. **Add the Telegram credential.** In n8n, go to Credentials, create a **Telegram API** credential, and paste the token. Select it in the Telegram Trigger node and every Telegram node.
4. **Replace the token in HTTP Request nodes.** The `HTTP Request` nodes call `https://api.telegram.org/bot<TOKEN>/sendMessage`. Put your own token in each of these URLs on your n8n instance only.
5. **Add the Google Calendar credential.** Create a **Google Calendar OAuth2 API** credential. On n8n Cloud use *Sign in with Google*. When self-hosting, create an OAuth client in Google Cloud Console, enable the Calendar API, add yourself as a test user, and use n8n's Redirect URL.
6. **Create the calendar.** In Google Calendar, create a new calendar named `Clinic Bookings`, then select it in the three calendar nodes: `Get many events`, `Get many events1`, and `Create an event1`.
7. **Set up the AI agent.** Add a credential for your chat model, connect it to the AI Agent node, and set the memory session ID to the patient's `chat_id`. Paste the clinic information (address, fees, working hours) and the restrictions (clinic information only, no booking, no medical advice) into the agent's system prompt.
8. **Activate the workflow** with the Publish button. Conversation state is only saved while the workflow is active, so test with the published workflow, not with *Execute workflow*.

## Configuration

These values are set inside the workflow's code nodes:

| Setting | Value |
|---|---|
| Working hours | 09:00 to 15:00 |
| Appointment length | 20 minutes |
| Days off | Friday and Saturday |
| Time zone | Africa/Cairo |
| Pending booking step expires after | 30 minutes |

## Usage

Open the bot in Telegram and send `/start`. Choose **Book** to see available days and times, or **Inquiry** for the clinic's address, fees, and hours. You can also type a question and the AI agent will answer it.

## Security

Never commit a bot token, API key, or Google credential to this repository. Keep them in n8n credentials only. If a token is ever exposed, revoke it immediately with `@BotFather` (`/revoke`) and generate a new one.

## Project documentation

All planning and design documents are in this repository:

- Project Planning & Management: Proposal, Project Plan, Task Assignment & Roles, Risk Assessment, KPIs
- Requirements Gathering
- System Analysis & Design
- Project Info Forms (weekly deliverables)

## Timeline

| Milestone | Due |
|---|---|
| Planning & Management, Requirements Gathering | 16 Oct 2026 |
| System Analysis & Design | 6 Nov 2026 |
| Implementation | 30 Nov 2026 |
| Final Presentation, Testing & Reports | 4 Dec 2026 |

## Team

| Member | Role |
|---|---|
| Ahmed Khaled Abdelfattah | Team Leader — planning, architecture, documentation |
| Ahmed Hossam Abdullah | n8n automation and integrations |
| Mohamed Alaa Ibrahim | Telegram bot logic and workflow build |
| Ziad Ahmed Fathy | Testing and QA |
| Ahmed Mossa Anwar | Documentation and presentation support |
