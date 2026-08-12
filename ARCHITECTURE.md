# One Fix, architecture

## Overview

A static front end and a single Cloudflare Worker. The Worker does everything that matters: it talks to Claude, runs the payment gate, sends the report, and stores state. The diagnosis logic never leaves the server.

## Stack

- **Front end:** static HTML on Cloudflare Pages. No CMS and no build step.
- **Backend:** one Cloudflare Worker with four jobs.
  - Proxy calls to Claude with structured JSON output
  - Create and verify Razorpay payments
  - Send email through Brevo
  - Store state in Cloudflare KV
- **Cost:** about five US cents of API spend per run.

## Request flow

1. A founder answers ten questions, one at a time, in a chat interface on the static page.
2. The page calls the Worker to create a Razorpay order. Razorpay handles checkout.
3. The Worker verifies the payment signature server side, then mints a one time access token.
4. The gated generate call carries that token. The Worker composes the prompt from the answers and calls Claude for a structured diagnosis.
5. The report is written to Cloudflare KV at an unguessable 32 character link, marked noindex, with no expiry. The token is consumed the moment the report sends.
6. The founder gets a summary email with a button to the full permanent report.

## Design choices

- **Server side prompt and taxonomy.** The system prompt and the diagnosis logic live in the Worker in one file. View source on the page shows nothing worth copying, and tuning the diagnosis never touches the page.
- **One payment, one report.** A token is minted only after a verified payment and consumed on send.
- **Permanent reports.** A paid artifact outlives the email, stored in KV at its own link.
- **Deploy verification as a URL.** A `/version` endpoint reports the running version and whether the payment gate is on.
- **A manual fallback.** If generation fails, the answers route to an internal inbox with reply to set to the founder, without consuming the token, so a retry still works and the report can be finished by hand.

## The payment gate, and the rule it taught

The gate flag has to agree in three places: a constant in the Worker, a variable on the tool page, and the live state of the sales page. If they disagree, the tool returns a 402 on every run. I learned that by bricking it live once. The three flags rule now sits at the top of the Worker file.

## Held back

The ten questions, the system prompt, the eight pattern logic, the scoring, and the fix list content are proprietary and are not in this repository.
