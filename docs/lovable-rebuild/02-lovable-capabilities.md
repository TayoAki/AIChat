# 02 — What Lovable can do (as of October 2026) and how the original stack maps to it

Researched from the official docs at docs.lovable.dev (sources at the bottom). Lovable changes fast, so recheck anything marked ⚠️ before relying on it.

## 1. Lovable in one minute

- **You build by chatting.** Three modes:
  - **Chat** discusses and debugs without changing code; cheap.
  - **Plan** researches your project and writes a reviewable plan to `.lovable/plan.md`; never edits code; costs one credit per message.
  - **Build** implements and verifies.
  - For big or risky stages, use **Plan → review → Approve**; approval switches to Build automatically.
- **Default app stack (new projects since 13 May 2026):** React with **TanStack Start (server-side rendering)**, Tailwind and shadcn/ui. Older projects are React + Vite SPAs and can be upgraded. → You do **not** get Next.js, but you get SSR, which helps the SEO blog.
- **Lovable Cloud** is the built-in backend, on by default and built on Supabase's open-source stack:
  - **Postgres database**: tables created from plain-language prompts, an SQL editor, RLS policies, daily backups. Migrations are saved in `drizzle/migrations/`; risky migrations are blocked unless you allow them.
  - **Auth**: email/password, magic link, **email OTP codes** (4–8 digits, configurable expiry), Google/Apple/Microsoft/SAML, password rules, leaked-password check.
  - **Storage** buckets.
  - **Realtime** (live updates).
  - **Edge functions**: serverless backend code that Lovable writes and deploys from your description.
  - **Secrets** for API keys, **Jobs** for scheduled tasks, **Logs**, usage.
  - Region is chosen once and can't be changed. There's no one-click migration between Cloud and your own Supabase, so pick early (Cloud is recommended here).
- **Lovable AI** gives AI inside your app through edge functions, with the key `LOVABLE_API_KEY` managed automatically.
  - Chat models include the Gemini 3.x Flash/Pro family (Gemini Flash is the default), the GPT-5.x/6 family and Claude models. There are also image, embedding, speech and voice models.
  - Streaming is supported via SSE. It's billed in Lovable credits. Rate limits return `429`; running out of credits returns `402`.
  - **Bring your own OpenAI key** is possible instead: store it as a secret and have Lovable call OpenAI from an edge function.
- **Payments:**
  - **Built-in payments** (Paddle or Stripe; Pro plan or higher): Lovable creates the account, products and checkout. They're digital-product oriented with one subscription per user.
  - **Your own Stripe account**: paste a restricted or secret key, stored as `STRIPE_SECRET_KEY`. Lovable writes edge functions for Checkout, subscription checks and the customer portal. Webhooks are optional.
  - ⚠️ **You can't use both in one project**, and **neither documents Stripe Connect**. We need Connect, so this build uses **your own Stripe account** and asks Lovable to call the Stripe Connect API from edge functions (the "integrate any API" route).
- **Email:**
  - **Lovable Emails** (Cloud "branded emails"): auth and app emails from your own domain, 50k/month included. Needs a paid plan and a domain whose DNS can hold subdomain `NS` records. ⚠️ **Transactional only; marketing and newsletters are not allowed.**
  - **Resend connector** (paste a `re_` key) supports transactional sends and marketing-style campaigns. You must verify your sending domain inside Resend.
- **Connectors** include WordPress (self-hosted), WordPress.com, Contentful, Storyblok (a headless CMS for the blog), Resend, Brevo, Mailgun, Klaviyo, Calendly, HubSpot, Slack and many more, plus "integrate any API".
- **Project Knowledge** (Project settings → Knowledge, up to 10,000 characters) is sent with every message. Use it for the product description, schema, style and rules (see [`03-project-knowledge.md`](03-project-knowledge.md)). Lovable also reads `AGENTS.md` / `CLAUDE.md` from a synced repo.
- **Other useful features:** version history (bookmark milestones), drafts, the preview toolbar for targeted edits, an SEO & AI-search review (sitemap, meta, OG, JSON-LD), a security view/scan, GitHub sync, custom domains, Figma import.

## 2. Stack mapping: original → Lovable

| Original | Lovable rebuild | Notes |
|---|---|---|
| Next.js App Router and server actions | TanStack Start app plus **Cloud edge/server functions** | Lovable decides where code runs. Keep secrets server-side. |
| Bun | n/a | Lovable manages the toolchain. |
| Clerk custom sign-up/in with email code | **Cloud Auth**: email and password, **email OTP verification code**, custom 3-step UI | No third-party widget, so no watermark problem. Set OTP length to 6 in Auth settings. Turn auto-confirm **off**. |
| Neon and Prisma | **Cloud Postgres**, with tables and RLS created by prompt | Prisma Studio → Cloud → Database view and SQL editor. |
| Clerk middleware public routes | Route guards in the app plus **RLS** in the database | RLS is the real security boundary. |
| Uploadcare | **Cloud Storage** buckets: `domain-icons`, `bot-icons`, `product-images` (public read); `chat-uploads` (written via a function) | 2 MB, png/jpg/jpeg validation stays. |
| Cloudways headless WordPress | **Option A (recommended):** a `blog_posts` table and an admin editor in the app, rendered with SSR. **Option B:** the WordPress (self-hosted) connector. | Option A removes a whole server. |
| Pusher | **Cloud Realtime** (Supabase Realtime) | Dashboard: listen to DB changes (RLS-protected). Widget: anonymous, so use broadcast per room or polling via a function. Fixes the hot-reload issue the video hit. |
| Nodemailer and Gmail app password | **Resend** (handoff alerts and campaigns) or Lovable Emails (handoff alerts only) | Lovable Emails can't send marketing campaigns. |
| OpenAI `gpt-3.5-turbo` | **Lovable AI** (default Gemini Flash model) from an edge function | Optional: your own OpenAI key as a secret. |
| `(complete)` / `(realtime)` keyword hacks | **Structured JSON output** from the model (reply, answered question, intent) | More reliable. The server builds the portal links, which fixes the doubled-link bug. |
| Stripe Connect with fake custom-account data | **Stripe Connect Express accounts and Account Links** via edge functions with your `STRIPE_SECRET_KEY` | Test mode onboarding has "use test data" shortcuts, so no fake data is needed. |
| Stripe PaymentElement on the portal | **Stripe Checkout** (embedded or hosted) on the connected account, called from an edge function | Simpler and PCI-safe. Use a destination charge with an optional platform fee. |
| Stripe PaymentIntent for plan upgrades | Stripe **Checkout in subscription mode** and the **customer portal** (Lovable's standard own-Stripe pattern) | Recurring billing, unlike the video's one-off payment. |
| Embed script and iframe with `postMessage` | Static `public/embed.js` that injects an iframe to `/chatbot?id=DOMAIN_ID`, plus `postMessage` resize | ⚠️ Iframe embedding of Lovable-hosted pages isn't documented. **Test it early** (Stage 9 includes the test). |
| next-themes and shadcn dark mode | Same pattern (Lovable defaults to shadcn and Tailwind tokens) | |
| Figma → Builder.io → code | Lovable Figma import, or paste screenshots into chat | |

## 3. Accounts, keys and costs you'll need

| Service | Needed for | When |
|---|---|---|
| **Lovable**, Pro plan recommended | Building; Cloud; custom domain; Lovable Emails | Stage 0 |
| **Stripe** (test mode) | Connect onboarding, product checkout, plan subscriptions. In Stripe → Settings → Connect, **turn on Connect and set your platform branding (name and icon) first**; the video hit an error without the icon. | Stage 14 |
| **Resend** with a verified sending domain | Handoff alert emails and email campaigns | Before Stage 10 |
| **A domain you own** (optional but recommended) | Custom app URL, Resend sender, Lovable Emails | Stage 21 |
| **GitHub** (optional) | Sync the code. Lovable's GitHub sync creates or links a repo; it can live next to this guide. | Any time |
| **OpenAI key** (optional) | Only if you'd rather not use Lovable AI credits | Stage 10 |

**Credit tips:**
- Use **Plan mode** for the big stages (schema, AI brain, realtime, Stripe) so you approve the approach before Build spends credits.
- Use **Chat mode** for questions and debugging.
- Make one stage per prompt and test before moving on.
- Bookmark versions after each working stage so you can revert freely.

## 4. Known gaps and risks (and how the prompts handle them)

1. **Stripe Connect isn't a turnkey Lovable feature.** The prompts spell out the exact Stripe API calls (`accounts.create` type `express`, `accountLinks.create`, Checkout with `transfer_data.destination` or `stripeAccount`).
2. **Anonymous widget users.** Visitors aren't logged in, so the widget and portal must never query tables directly. They call **public edge functions** that check `domain_id` plus a secret `visitor_token` (widget) or the unguessable `customer_id` (portal).
3. **Iframe embedding** of the published app on third-party sites. Test it as soon as the `/chatbot` route exists (Stage 9). If it's blocked, ask Lovable in Chat mode how to allow framing for that route, or contact Lovable support.
4. **Realtime for anonymous visitors.** Broadcast per room, with polling every 3 seconds via an edge function as a fallback.
5. **Email marketing compliance.** Add an unsubscribe link and an `unsubscribed` flag. Lovable Emails forbids marketing, so use Resend.
6. **Credit and usage costs.** AI replies, functions and emails all draw on Lovable run credits or your provider bills. Keep prompts short and history windows small (last 20 messages).

## Sources

- [Lovable docs index (llms.txt)](https://docs.lovable.dev/llms.txt)
- [Lovable Cloud](https://docs.lovable.dev/features/cloud) · [Database](https://docs.lovable.dev/features/database) · [Edge functions](https://docs.lovable.dev/features/edge-functions) · [Email authentication](https://docs.lovable.dev/features/email-auth)
- [AI features for your app (Lovable AI)](https://docs.lovable.dev/features/ai)
- [Add payments (built-in)](https://docs.lovable.dev/features/payments) · [Connect your own Stripe account](https://docs.lovable.dev/integrations/stripe)
- [Branded emails](https://docs.lovable.dev/features/custom-emails) · [Resend connector](https://docs.lovable.dev/integrations/resend)
- [WordPress (self-hosted) connector](https://docs.lovable.dev/integrations/wordpress)
- [SEO and AI search](https://docs.lovable.dev/features/seo-aeo) · [Upgrade to TanStack Start](https://docs.lovable.dev/features/upgrade-to-tanstack-start) · [Hosting](https://docs.lovable.dev/features/hosting)
- [Prompting best practices](https://docs.lovable.dev/prompting/prompting-one) · [Knowledge](https://docs.lovable.dev/features/knowledge) · [Plan mode](https://docs.lovable.dev/features/plan-mode)
