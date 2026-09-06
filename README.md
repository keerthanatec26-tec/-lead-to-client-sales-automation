# Lead-to-Client Sales Automation Pipeline

AI-powered lead qualification and outreach system built in n8n. When a lead submits a website contact form, this pipeline automatically researches the company, scores the lead's buying intent, sends a personalized outreach email, logs everything to a CRM, and alerts the sales team in real time — work that typically takes a human 15–30 minutes per lead, done in under a minute.

---

# The Problem

Most businesses handle inbound leads manually: someone checks the inbox, researches the company, decides if it's worth chasing, writes a reply, and updates a spreadsheet. By the time that happens, a serious buyer may have already gone to a competitor who responded faster. Speed-to-lead is one of the strongest predictors of whether an inquiry converts into a customer — this pipeline closes that gap.

---

# Architecture
<img width="1792" height="717" alt="architecture-diagram" src="https://github.com/user-attachments/assets/95c0c528-3c03-49b8-be0e-f4df92979ad2" />


---

# What It Does

1. **Captures leads instantly** via a webhook — any website form can POST directly to it.
2. **Normalizes the data** — trims whitespace, lowercases emails, handles missing fields.
3. **Checks email type** — company domain vs. personal (Gmail/Yahoo/Outlook), an early signal of lead quality.
4. **Researches the company** — fetches the lead's company website and uses a local LLM (via Ollama) to extract what the business does, grounded strictly in the fetched content to avoid hallucination.
5. **Scores the lead** — an AI agent classifies the lead as **Hot**, **Warm**, or **Cold** against explicit, checkable criteria (company email + seniority + urgency = Hot; a specific product question = Warm; vague or personal-email inquiries = Cold), returning strict JSON so the workflow can route on it automatically.
6. **Writes a personalized email per tier** — Hot leads get a direct, urgency-matched email referencing their specific need; Warm leads get a softer, no-pressure, informational email.
7. **Sends the email automatically** via Gmail.
8. **Logs every lead to a CRM** (Google Sheets) — date, contact details, score, reasoning, and outreach status, regardless of tier.
9. **Alerts the sales team on Slack** — but only for Hot leads, so the channel stays high-signal and reps respond within minutes, not hours.
10. **Monitors itself** — a dedicated error-handling workflow catches failures anywhere in the pipeline (a bad AI response, a failed send, an API timeout) and posts an immediate alert, instead of a lead silently falling through the cracks.

---

# Tech Stack

- **n8n** (self-hosted) — orchestration
- **Ollama, running qwen2.5:7b locally** — company research summarization and lead scoring
- **Gmail API** — automated outreach
- **Google Sheets API** — CRM logging
- **Slack Incoming Webhooks** — real-time sales notifications

---

# Demo



*Click the image above to watch a full walkthrough of the pipeline in action.*

---

# Screenshots

**Real-time Slack alert for a hot lead:**
<img width="1902" height="911" alt="slack-alert" src="https://github.com/user-attachments/assets/8b5ee618-ee4d-49d9-ae3b-99e8b1961f69" />


**CRM logging in Google Sheets:**
<img width="1897" height="767" alt="crm-sheet" src="https://github.com/user-attachments/assets/b79550e7-38dc-40b3-ac55-f19472f9d04c" />


**AI-personalized outreach email, actually received:**
<img width="738" height="1600" alt="sample-email(HOT)" src="https://github.com/user-attachments/assets/563bf4d9-d26e-4fe3-a059-ccab6052a8c3" />
<img width="738" height="1600" alt="sample-email(WARM)" src="https://github.com/user-attachments/assets/f74494d3-6af4-49b7-b5d9-6816b20936fd" />


---

# Key Engineering Details

- **Grounded AI outputs** — both the research and scoring prompts explicitly instruct the model not to guess beyond the provided data, reducing hallucination — a real risk when this kind of output reaches a client's CRM or inbox.
- **Cross-node data references** — n8n's AI Agent nodes only pass forward their own output, not upstream fields. This workflow uses `$('NodeName').item.json.field` references throughout to reliably carry the original lead data across every AI step.
- **Structured decision-making** — the scoring step forces strict JSON output specifically so the result can drive automatic routing (via a Switch node), rather than requiring a human to read and interpret free text.
- **Graceful degradation** — a personal-email lead skips company research entirely (no website to check) and flows directly to scoring, with the prompt handling the missing research field explicitly rather than breaking.
- **Production-ready error handling** — the main workflow is linked to a separate Error Workflow, so any node failure — a malformed AI response, a failed API call — triggers an immediate Slack alert rather than failing silently.

---

# Business Value

This is packaged as a real, sellable service — the kind of "Complete Sales Automation System" automation agencies typically charge ₹30,000–₹1,00,000+ to build. It's built to be adapted per client: swapping the CRM node for a client's actual CRM (HubSpot, Zoho, Pipedrive), tuning the scoring criteria to their specific business, and pointing the webhook at their live website form.

---

*Built as a capstone project in a self-directed n8n automation learning path, following 8 prior projects covering webhooks, AI agents, structured outputs, and API integrations.*
