# Rebuild "Corinna AI" (Web Prodigies) in Lovable

A guide to rebuilding the 9-hour tutorial [*SAAS AI Chatbot – Email Marketing, Next.js 15, Clerk, Neon, Uploadcare, Cloudways, Bun, Stripe, Pusher*](https://www.youtube.com/watch?v=9pCsyBlpmrc) in **Lovable**, without writing the Next.js code by hand.

The working name in the prompts is **AIChat**. Search and replace it with your own brand.

## What you're building (one paragraph)

A multi-tenant SaaS where businesses sign up, add their website domains, and paste one script tag to get an **AI sales-rep chatbot** on their site. The bot:
- greets visitors and captures their **email** (a lead);
- asks the owner's **qualifying questions** naturally;
- sends a link to a **booking page** or a **Stripe payment page** for the business's products;
- **hands off to a human** (and emails the owner) when the visitor is frustrated.

Owners get a dashboard with live chat takeover, appointments, products with Stripe Connect payouts, email campaigns to captured leads, metrics, a light/dark theme and paid plans (Standard / Pro / Ultimate). The landing page has pricing and an SEO blog.

## How to use this guide

| Step | File | What it's for |
|---|---|---|
| 1 | [01-video-breakdown.md](01-video-breakdown.md) | The whole transcript, understood: features, original architecture, data model, AI logic, chapter map, what the video left unfinished |
| 2 | [02-lovable-capabilities.md](02-lovable-capabilities.md) | What Lovable can do today (Cloud, Lovable AI, payments, email, connectors), how each original tool maps to it, the accounts you need, risks |
| 3 | [03-project-knowledge.md](03-project-knowledge.md) | Paste into **Project settings → Knowledge** after Stage 1 (about 7k of the 10k-character limit) |
| 4 | [04-build-prompts.md](04-build-prompts.md) | **22 copy-paste prompts, one per stage**, each with a "Done when" checklist |
| 5 | [05-testing-and-debugging.md](05-testing-and-debugging.md) | End-to-end test script, debugging prompt templates, and the video's bugs and how this build avoids them |

## Build order at a glance

```
0 Accounts → 1 Landing → 2 Schema + RLS → 3 Auth (OTP) → 4 Dashboard shell → 5 Domains
→ 6 Settings → 7 Domain settings + embed snippet → 8 Bot training → 9 Widget + iframe test
→ 10 AI brain + handoff email → 11 Inbox + realtime → 12 Booking portal → 13 Appointments
→ 14 Stripe Connect → 15 Products → 16 Payment portal → 17 Plan subscriptions
→ 18 Email marketing → 19 Dashboard metrics → 20 SEO blog → 21 Hardening + publish
```

**Milestones worth bookmarking in Lovable's version history:**
- After Stage 9: the widget is embeddable.
- After Stage 11: the AI and live chat work end to end.
- After Stage 16: money flows.
- After Stage 21: ready to launch.

## Key decisions (and why)

- **Lovable Cloud instead of Clerk + Neon + Uploadcare + Pusher.** One backend gives you auth (with email OTP), Postgres with RLS, storage, realtime and edge functions, so there are no extra accounts or SDKs.
- **Lovable AI instead of OpenAI.** No key to manage. Swap in your own OpenAI key as a secret if you prefer provider billing.
- **Structured JSON from the model instead of `(complete)` / `(realtime)` keywords.** The server decides links and handoff, so the model can't break URLs.
- **Your own Stripe key (not Lovable built-in payments).** Built-in payments don't cover **Stripe Connect**, and the two can't be mixed in one project.
- **Resend for email.** Lovable's built-in Emails are transactional-only, and campaigns are marketing email.
- **Built-in blog table instead of a Cloudways WordPress server.** New Lovable apps render on the server (TanStack Start), so the blog is SEO-friendly. The WordPress connector is still an option.
- **Widget and portal go through public edge functions.** Anonymous visitors never touch tables directly. They're validated by domain ID plus a secret visitor token or the customer ID.

## Notes

- The email marketing part was accidentally cut from the original video. Stage 18 designs it from the demo.
- Prices and limits (Standard free with 1 domain; Pro $15 with 5; Ultimate $35, unlimited) are placeholders. Edit `plans.ts` and the Knowledge text together, and ask Lovable to update the mirrored limits in the domain-limit trigger and edge functions.
- Lovable features and model names change often. Recheck [docs.lovable.dev](https://docs.lovable.dev) for anything flagged ⚠️ in file 02.
