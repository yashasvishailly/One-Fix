# One Fix, architecture

## Overview

Current flow checked against the private source on 22 October 2026. This is a
source-backed case study, not a live deployment verification.

A static front end and a single Cloudflare Worker. The Worker does everything that matters: it generates the diagnosis, runs the payment gate, sends the report, and stores state. The diagnosis logic never leaves the server.

## Stack

- **Front end:** static HTML on Cloudflare Pages. The repository build script prepares the published output; there is no CMS.
- **Backend:** one Cloudflare Worker with four jobs.
  - Generate structured diagnosis output
  - Create and verify Razorpay payments, verify Dodo webhooks, and accept authorized redemption
  - Send email through Brevo
  - Store state in Cloudflare KV
- **Shared claim storage:** durable claims protect concurrent redemption and delivery operations alongside the KV session/report store.
- **Cost:** depends on the configured model, answer length, follow-ups, and retries. No fixed per-report estimate is asserted here.

## Request flow

1. The founder starts at the sales page and completes checkout through a supported payment rail, or uses an authorized redemption code.
2. The Worker verifies payment or redemption on the server before issuing diagnostic access.
3. The tool checks that access and retrieves its base questions from the Worker. When needed, it asks a short follow-up round based on the answers.
4. The generate call carries the session token. The Worker composes the prompt and returns a structured diagnosis.
5. The report is stored at an unguessable link and the delivery flow records the session's report URL. Returning with a completed session can reopen that report.
6. Email delivery sends a summary and a link to the saved report. Failure and retry behavior is handled by the server and the tool rather than treating a temporary access-check outage as proof of unpaid access.

## Design choices

- **Server side prompt and taxonomy.** The system prompt and the diagnosis logic live in the Worker in one file. View source on the page shows nothing worth copying, and tuning the diagnosis never touches the page.
- **One payment, one report.** Diagnostic access is issued after verified payment or authorized redemption. The server checks session status and quotas; completed access points back to its saved report.
- **Permanent reports.** A paid artifact outlives the email, stored in KV at its own link.
- **Deploy verification as a URL.** A `/version` endpoint reports the running version and whether the payment gate is on.
- **A manual fallback.** If generation fails, the answers route to an internal inbox with reply to set to the founder, without consuming the token, so a retry still works and the report can be finished by hand.

## The original payment-gate integration lesson

The gate flag has to agree in three places: a constant in the Worker, a variable on the tool page, and the live state of the sales page. If they disagree, the tool returns a 402 on every run. I learned that by bricking it live once. That rule records the original integration failure. Current deployment work should use the maintained source, build instructions, and tests rather than reproduce a historical configuration from this case study.

## Held back

The ten questions, the system prompt, the eight pattern logic, the scoring, and the fix list content are proprietary and are not in this repository.
