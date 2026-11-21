# AI PRD · Juno

> Module 3 · Write an AI PRD Engineers Can Build from. The AI product requirements doc specifying all six harness surfaces, built with the **M3 · AI PRD Builder** (archetype and starting settings from the **M3 · Harness Architecture Decider**). Paste the tool's markdown over this file.

## Problem & user

Small and mid-size business (SMB) customers run into trouble when they set up Knox and buy licenses: they aren't sure which product to pick, where to start, what the price is, or how to activate. They write this down as free-text feedback.

Every week a Program Manager has to turn about 50 of these messages into a short, ranked list of what to fix next — without missing anything and without ever exposing a customer's personal details. Today this is done by hand. It is slow, easy to get wrong, and risky because the raw feedback contains names, emails, and phone numbers.

Juno is an assistant that does this weekly triage for them.
_____

## The harness

| Surface | Specification |
|---|---|
| 01 Context | The newest weekly feedback file, plus Knox product and policy documents. |
| 02 Tools | Read the file, remove personal details, sort issues, score priority, draft the briefing, raise a ticket. |
| 03 Loop | Runs once a week. Stops and asks a person whenever a check fails or file is not present. |
| 04 Memory | Keeps weekly totals for trend lines. Never keeps personal details. Keep last week(s) issues to compare |
| 05 Permissions | Reading and drafting happen on their own. Creating a ticket needs a human click. Sending anything out is not supported |
| 06 Verification | Before anything is shown, Juno checks the file, the personal-data removal, the counts, and the priority. |

## 01 Context · Data Requirements

- **Required sources:**
  - The newest weekly feedback file in `D:\Excercise\`, named `Customer issue week <N>.xlsx` (two columns: an ID and the customer's message).
  - Knox product information (Suite, Manage, Configure, Mobile Enrollment) so issues can be sorted correctly.
  - Knox help and policy documents (buying and billing, license management, admin setup, device enrollment, trials, account security) used to suggest fixes.
  - Last week's totals, kept in Memory, for the "vs last week" comparison.
- **Deliberate exclusions:**
  - Older weekly files (except last week's totals). Reason: the rule is to work only on the newest file.
  - Any customer database, billing system, or other store of personal data. Reason: keep personal details out.
  - The open internet. Reason: answers must come from the file and Knox's own documents.
- **SLA of truth:** The file is updated every Sunday evening. Juno picks up that change on its next weekly run. So the picture is accurate to within one week — a change made mid-week is not seen until the next run.
- **When a source is unreachable:** Stop and say so clearly. Juno must never quietly fall back to an older file.


## 02 Tools · System Capabilities

- **The verb list** (and what each one touches):
  - `find_files` — look in the folder (read only)
  - `pick_newest` — choose the newest file (read only)
  - `read_file` — open the chosen file (read only)
  - `remove_personal_details` — strip names, emails, phones, ZIP, payment info (read and clean)
  - `sort_issues` — group each issue into one feature area (read only)
  - `flag_duplicates` — mark repeated issues (read only; it flags, never deletes)
  - `score_priority` — turn an issue count into a priority (read only; fixed formula)
  - `build_briefing` — prepare the dashboard view (draft)
  - `create_ticket` — raise a sprint ticket for the Program Manager (write)
  - `save_weekly_totals` — store this week's totals for next time (write)
- **Deliberate omissions:**
  - No "send email" or "notify" tool. Reason: sending something out of the system can't be taken back, so a person does that, not Juno.
  - No tool to edit or delete the original feedback rows. Reason: the source file must stay untouched.
  - No automatic ticket creation. Reason: a person decides what becomes a ticket.

## 03 Loop · AI Costs & Latency

- **Turn ceiling:** This runs as one weekly job, not a back-and-forth chat. The AI does at most **3 passes** over the batch of ~50 messages: one to sort, one to summarise, one to suggest fixes. Creating a ticket is a human click, not an extra AI step.
- **Escalation trigger:** Juno stops and hands back to a person if any check fails (wrong or missing newest file, a personal detail slips through, the numbers don't add up), if it isn't confident how to sort an issue, or if an issue doesn't fit any known feature area.
- **Latency and cost target:** A full weekly run finishes in **under a minute** and costs **about $0.50 or less** for ~50 messages (to be confirmed against the chosen model). The dashboard itself opens instantly.

## 04 Memory · Data Requirements

- **What persists, at what scope:**
  - Weekly totals (overall and per feature area) — shared across the team, kept for about the last 8 weeks for trend lines.
  - The priority settings (the count ranges) — a shared team setting.
  - Which feature areas already have a ticket this week — kept only for the current week.
  - The Knox policy reference list — shared, long-lived.
- **Expiry:**
  - Raw feedback and any personal details — **kept for zero time** (never stored).
  - Weekly totals — drop off after about 8 weeks.
  - This week's ticket list — cleared when the new week starts.
- **Write rules:** A person always outranks Juno. The Program Manager can change a priority, move an issue to a different area, or cancel a ticket, and Juno never overwrites what a person decided.

## 05 Permissions · AI Risks & Mitigations

| Side-effect class | Tier    |
|---                |---      |
| read              | auto    |
| draft             | auto    |
| write             | confirm |
| send              | blocked |

**Justification:** Reading the file and drafting the briefing only look at the local file and Knox documents and produce nothing that leaves the system, so they are safe to run on their own and easy to redo. Writing means creating a ticket or saving totals; a wrong ticket is easy to close but still creates noise, so it only happens when a person clicks "Create Ticket". Sending anything outside the system is turned off completely in this version — it can't easily be undone and isn't needed for triage.

## 06 Verification · AI Testing & Measurement

- **The check before output ships:** Before showing anything, Juno confirms all of the following:
  1. The chosen file is the newest one, has the right name, and has the two expected columns.
  2. No personal details (name, email, phone, ZIP, payment) appear anywhere in the briefing, a ticket, or memory.
  3. The numbers add up — the feature areas total the distinct-issue count, and the percentages are close to 100%.
  4. Each ticket's priority matches the count formula exactly.
  5. Duplicates are flagged and left out of the counts.
  The bar: every check must pass. The personal-details check is a hard stop.

- **Failure behaviour:** If any check fails, Juno stops and tells the person exactly which check failed and where (which file or row). It never shows a half-finished or unchecked briefing, and it never quietly drops the result.

## Eval plan

_Stub. Module 5 fills this in: the human eval rubric in `05-evals/human-rubric.md`, with its pass bar._

## Out of scope

Everything not in the verb list, and everything marked `blocked`, is a deliberate decision, not a gap:
- Sending emails or notifications.
- Creating tickets automatically, with no human click.
- Editing or deleting the original feedback.
- Looking at older weeks issue-by-issue (only last week's totals are used).
- Issues that are not about Knox onboarding or licensing.
- Connecting to a live customer, billing, or CRM system.
