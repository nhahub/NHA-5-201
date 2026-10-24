# Smart Clinic AI Assistant

**DEPI — Digital Egypt Pioneers Initiative | Track: AI & Data Science**

An n8n-based automation system for a single-doctor clinic. Patients talk to a Telegram bot to ask about the clinic and book appointments, while scheduled workflows handle reminders, the daily financial report, and post-visit feedback.

**Try the bot:** <https://t.me/YOUR_BOT_USERNAME> *(works only while the n8n workflow is active)*

-----

## Features

|Module                   |What it does                                                                                                                                    |Status                 |
|-------------------------|------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------|
|1. Inquiries & booking   |Button-driven Telegram menu (no LLM). Answers address, fees, and working hours, and books appointments against real Google Calendar availability|Working (first version)|
|2. Reminders             |Sends an interactive reminder 24 hours before each appointment: Confirm / Reschedule / Cancel                                                   |In progress            |
|3. Daily financial report|Computes daily revenue and case count in code (not by the LLM) and emails the doctor via Gmail                                                  |In progress            |
|4. Patient feedback      |Requests feedback after the visit; an LLM classifies it as Positive, Neutral, or Negative                                                       |In progress            |

**Design principle:** decisions that must be exact (availability, bookings, money) are made in code. The LLM is used only for feedback sentiment classification.

## How booking works

1. The patient taps **Book** and the bot offers the next working days (Friday and Saturday are skipped).
1. After the patient picks a day, the workflow reads that day’s events from Google Calendar.
1. Code generates 20-minute slots between 09:00 and 15:00 (Africa/Cairo time), removes past slots and slots that overlap existing events, and shows the rest as buttons.
1. The bot asks for the patient’s name and age, validates them, and shows a summary to confirm.
1. On confirmation, the slot is checked **again** against the calendar. If it is still free, an event is created; otherwise the patient is told the slot was just taken. This second check prevents double booking.

## Tech stack

- **n8n** — workflow automation engine
- **Telegram Bot API** — patient interface
- **Google Calendar API** — availability and appointments
- **Google Sheets API** — data store for bookings, ledger, and feedback
- **Gmail API** — daily report delivery
- **LLM API** — feedback sentiment classification only

## System requirements

- An n8n instance (n8n Cloud, or self-hosted) reachable over a **public HTTPS URL**. The Telegram Trigger cannot receive messages from a plain `localhost`.
- A Telegram account, to create a bot with `@BotFather`.
- A Google account with Google Calendar (and later Sheets and Gmail).
- Optional, for the feedback module: an API key for an LLM provider.

## Installation and setup

1. **Import the workflow.** In n8n, open Workflows, choose **Import from file**, and select the workflow JSON from the `workflows/` folder.
1. **Create your Telegram bot.** In Telegram, message `@BotFather`, send `/newbot`, and copy the token it gives you.
1. **Add the Telegram credential.** In n8n, go to Credentials, create a **Telegram API** credential, and paste the token. Select it in the Telegram Trigger node and every Telegram node.
1. **Replace the token in HTTP Request nodes.** The `HTTP Request` nodes call `https://api.telegram.org/bot<TOKEN>/sendMessage`. Put your own token in each of these URLs on your n8n instance only.
1. **Add the Google Calendar credential.** Create a **Google Calendar OAuth2 API** credential. On n8n Cloud use *Sign in with Google*. When self-hosting, create an OAuth client in Google Cloud Console, enable the Calendar API, add yourself as a test user, and use n8n’s Redirect URL.
1. **Create the calendar.** In Google Calendar, create a new calendar named `Clinic Bookings`, then select it in the three calendar nodes: `Get many events`, `Get many events1`, and `Create an event1`.
1. **Activate the workflow** with the Publish button.

## Configuration

These values are set inside the workflow’s code nodes:

|Setting           |Value              |
|------------------|-------------------|
|Working hours     |09:00 to 15:00     |
|Appointment length|20 minutes         |
|Days off          |Friday and Saturday|
|Time zone         |Africa/Cairo       |

## Usage

Open the bot in Telegram and send `/start`. Choose **Book** to see available days and times, or **Inquiry** for the clinic’s address, fees, and hours.

## Security

Never commit a bot token, API key, or Google credential to this repository. Keep them in n8n credentials only. If a token is ever exposed, revoke it immediately with `@BotFather` (`/revoke`) and generate a new one.

## Project documentation

All planning and design documents are in this repository:

- Project Planning & Management: Proposal, Project Plan, Task Assignment & Roles, Risk Assessment, KPIs
- Requirements Gathering
- Project Info Form

## Timeline

|Milestone                                    |Due        |
|---------------------------------------------|-----------|
|Planning & Management, Requirements Gathering|16 Oct 2026|
|System Analysis & Design                     |6 Nov 2026 |
|Implementation                               |30 Nov 2026|
|Final Presentation, Testing & Reports        |4 Dec 2026 |

## Team

|Member                  |Role                                               |
|------------------------|---------------------------------------------------|
|Ahmed Khaled Abdelfattah|Team Leader — planning, architecture, documentation|
|Ahmed Hossam Abdullah   |n8n automation and integrations                    |
|Mohamed Alaa Ibrahim    |Telegram bot logic and workflow build              |
|Ziad Ahmed Fathy        |Testing and QA                                     |
|Ahmed Mossa Anwar       |Documentation and presentation support             |