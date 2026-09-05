# One Fix

A paid founder diagnostic. Ten questions in a chat interface, about a minute of wait, and a scored nine section report comes back that names the execution pattern a company is stuck in and hands over a prioritized fix list. Built for structured diagnosis, running on a single Cloudflare Worker. 

Live: https://yashasvishailly.com/diagnose
Teardown: https://yashasvishailly.com/blog/the-one-fix-breakdown

## What I built

Every consulting engagement I run opens the same way. A founder names a symptom, and the first hour is me walking them from the symptom to the actual cause. That walk has a fixed shape, so I built it.

It started in April 2026 as a plain form that emailed me answers I diagnosed by hand. In May I rebuilt it around a tighter decision flow, and the wait from answering to knowing dropped from days to about a minute. Since then it has grown into a scored nine section diagnosis across execution, operations, decision making, people, and org design, with an eight pattern taxonomy underneath that names the dominant pattern and attaches a confidence level.

I built and debugged every piece myself: the diagnosis engine, the payment gate, the email, the permalinks, and the failure paths.

## The eight patterns

The engine reads all ten answers together and names the dominant execution pattern from a taxonomy I built out of eight years of watching companies get stuck. A few of them: Founder Gravity, where everything routes through the founder. The Orphaned Middle, a middle layer that was hired and never given real authority. The Consensus Tax, where decisions wait for an agreement that never comes. The full set and the reasoning are in the teardown.

Naming the pattern mattered more than the score. Founders argue with a score. They go quiet when a pattern gets named, because a name means someone has seen it before, which means it's real and fixable.

## What I learned shipping it

The diagnosis engine, the part people would call the AI, was maybe a fifth of the work. The other four fifths was payments, access tokens, email templates, and failure paths. Every hard bug lived on an integration seam: page to Worker, Worker to payments, Worker to email, build to deploy. One deploy bricked the tool live, a 402 on every run, because three payment flags disagreed. That produced a rule now written at the top of the Worker file.

A few decisions I'm glad I made:

- **Permalinks for a paid artifact.** Every report is stored and served at its own unguessable link with no expiry, so the thing you bought outlives the email.
- **A one time access token,** minted only after a server side payment check and consumed the moment the report sends. One payment, one report.
- **A `/version` endpoint,** so I can verify which build is live from my phone.
- **A server side prompt.** The taxonomy and diagnosis logic live in the Worker. View source on the page shows nothing worth copying.

## What it runs on

- Static HTML on Cloudflare Pages
- One Cloudflare Worker: proxies Claude with structured JSON output, creates and verifies Razorpay payments, sends email through Brevo, and stores state in Cloudflare KV

## What's in this repository

A case study and the system architecture. See `ARCHITECTURE.md`. The product stays private. The ten questions, the system prompt, the scoring, and the fix list content are what people pay for, so they are not here.

## Who built it

Yashasvi Shailly, an operator with eight years inside early to growth stage startups. One Fix is the first hour of that consulting conversation, compressed to about a minute and built for structured diagnosis.
