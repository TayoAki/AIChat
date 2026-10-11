# 01 — Understanding the video

**Video:** *SAAS AI Chatbot – Email Marketing, Next.js 15, Clerk, Neon, Uploadcare, Cloudways, Bun, Stripe, Pusher* (Web Prodigies, May 2024, about 9 hours 8 minutes).
**Product built:** "Corinna AI" (the creator pronounces it "Kina"), an AI sales-rep chatbot that businesses embed on their own websites.

I read the full transcript (about 117,000 words) end to end. The video covers 55 chapters. This file condenses them into what was built, how it works, and what was left unfinished. Ads, community promotion, the SEO talk and the sales talk are summarised in a single line each at the end.

---

## 1. What the product does (from the demo, 0:41–13:14)

| Area | What the user sees |
|---|---|
| **Landing page** | Minimal hero, three pricing plans (Standard / Pro / Ultimate), and SEO blog posts pulled from a headless CMS. |
| **Sign up / sign in** | Custom multi-step onboarding without Clerk's watermark: (1) choose account type ("I own a business" / "I'm a student"), (2) name, email, confirm email, password, confirm password, (3) 6-digit email OTP. A progress bar shows the step. |
| **Dashboard** | Metric cards for potential clients, pipeline value, appointments and total sales. Plan-usage progress bars for email credits, domains and contacts. A recent-transactions list. |
| **Domains** | The "+" in the sidebar opens a bottom drawer to add a domain (name and icon). Each domain gets one chatbot. Plan limits apply: Standard allows 1 domain. |
| **Domain settings** | Rename the domain, copy the **embed code snippet**, customise the bot icon and greeting (Pro and Ultimate only), delete the domain. |
| **Bot training** | Two tabs. **Help Desk** holds FAQ question-and-answer pairs that visitors can browse. **Questions** holds "filter questions" that the AI must get answered during the chat to qualify the lead. |
| **Products** | Add a product (name, image, price) per domain. Products are sold on the customer portal. |
| **The chatbot (on any website)** | A floating bubble that opens a chat window with *Chat* and *Help Desk* tabs and image upload. The AI greets the visitor, gets their email first, recognises returning customers by email, then asks the filter questions naturally. |
| **Smart links** | When the visitor wants to book, the bot sends a **portal link to a booking page**. When they want to buy, it sends a link to a **payment page**. Unanswered filter questions are shown pre-filled on the portal so the visitor can finish them. |
| **Human handoff** | If the visitor gets frustrated or asks for a human, the bot says it will connect them to support, switches the room to **real-time mode**, and **emails the business owner** once. |
| **Conversations inbox** | The owner sees chat rooms per domain with an "urgent" marker. They can open a room, toggle real-time mode, and chat live with the visitor. Messages and images appear instantly on both sides. |
| **Customer portal** | Step 1: answer any missing questions. Step 2: pick a date and a 30-minute slot (slots already booked are disabled) **or** pay for the domain's products with a custom Stripe form. Step 3: thank-you page. |
| **Appointments** | A table of all bookings and a "today" column. |
| **Integrations** | "Connect Stripe" uses Stripe Connect, so each business gets paid directly for its products. |
| **Email marketing** | A leads table (email, source domain, a "View" button for the answers), create campaigns, add selected leads, write an email template, send. Email credits come from the plan and go down with each email sent. |
| **Settings** | Current plan with its features, an upgrade modal (Stripe subscription), a light/dark/system theme picker, and a change-password form. |
| **General** | Fully responsive, collapsible sidebar, dark mode. |

**The core loop:** a visitor chats → the bot captures the email (now a lead) → asks the qualifying questions → sends a booking or payment link → the owner follows up from the inbox, the appointments page or an email campaign.

---

## 2. Original tech stack and why it was used

| Concern | Original choice | Role in the app |
|---|---|---|
| Framework | Next.js 14/15 App Router, Server Actions, TypeScript, Bun | Pages, routing, all backend logic as server actions |
| UI | Tailwind, shadcn/ui (`add --all`), lucide-react, next-themes, Plus Jakarta Sans | Components, dark mode |
| Auth | Clerk v4 with **custom** sign-in/up pages (`useSignUp`, `useSignIn`, email-code verification) | Accounts without a third-party-branded UI |
| Database | Neon serverless Postgres and Prisma ORM | All data |
| Forms | react-hook-form, zod, @hookform/error-message, a reusable **FormGenerator** component | Every form is driven by config arrays in `constants/` |
| File uploads | Uploadcare (`@uploadcare/upload-client`) | Domain icons, bot icons, product images, chat images |
| Blog / CMS | Cloudways-hosted **headless WordPress** (WP REST API with the Custom Post Type UI plugin), axios, html-react-parser | SEO blog on the landing page and `/blogs/[id]` |
| Real-time | Pusher (pusher and pusher-js) | Live owner ↔ visitor chat |
| AI | OpenAI `gpt-3.5-turbo` chat completions | The sales bot |
| Email | Nodemailer and a Gmail app password | Owner alert on handoff, and campaign emails |
| Payments | Stripe Connect (a custom account filled with test data to skip onboarding), Payment Element, and Stripe subscriptions | Businesses sell products; the platform sells plans |
| Embedding | A script snippet that injects an **iframe** to `/chatbot`, with `postMessage` between parent and iframe | Bot appears on any website |

**Folder pattern used:** `actions/` (server actions per feature), `hooks/` (one hook per feature that holds form, state and calls), `context/` (auth-step context, chat context), `schemas/` (zod), `constants/` (form field configs, menus, pricing, time slots), `components/` (forms, sidebar, infobar, chatbot, portal, etc.).

---

## 3. Data model in the original

Prisma models mentioned in the video, with their relationships:

- **User**: full name, Clerk ID, type (owner/student), Stripe Connect ID → has many **Domain**, has one **Billings**, has many **Campaign**
- **Billings**: plan (`STANDARD | PRO | ULTIMATE`), credits
- **Domain**: name, icon → has one **ChatBot**, many **HelpDesk**, many **FilterQuestions**, many **Customer**, many **Product**
- **ChatBot**: welcome message, icon, background colour, text colour, help-desk toggle
- **HelpDesk**: question, answer
- **FilterQuestions**: question, answered
- **Customer**: email → many **CustomerResponses**, many **ChatRoom**, many **Bookings**
- **CustomerResponses**: question, answered
- **ChatRoom**: live (real-time on/off), mailed (owner already emailed) → many **ChatMessage**
- **ChatMessage**: message, role (`user` / `assistant`), seen, createdAt
- **Bookings**: date, slot, email, domainId
- **Product**: name, price, image
- **Campaign**: name, customers[], template

---

## 4. How the AI bot works in the original (5:30:07–6:45:05)

1. **Widget mount.** The iframe loads `/chatbot`. The parent page sends the domain ID via `postMessage`. The bot loads the chatbot settings (welcome message, icon, help desk) and asks the parent to resize the iframe on open and close.
2. **Every visitor message** goes to one server action (`onAiChatBotAssistant`) along with the full chat history held in the browser.
3. **The server tries to extract an email** from the message with a regex.
   - **Email found:** look up the customer by email on this domain. If they're new, create the customer, a chat room, and one `CustomerResponses` row per filter question. Reply with a welcome-aboard message.
   - **Customer known:** if the room is `live`, just store the message and push it over Pusher; no AI. Otherwise call OpenAI with a system prompt:
     - "Here are the questions you must ask; progress the conversation with them like a natural conversation (not a sales script)."
     - "When you ask one of these questions, append the keyword **`(complete)`**." The next visitor reply is then saved as the answer to the first unanswered question.
     - "If the visitor is inappropriate or out of context, say you'll get a real person, and append **`(realtime)`**." The server then flips the room to `live`, emails the owner via Nodemailer (only once, using `mailed`), and stops AI replies.
     - "If they agree to book, give them `…/portal/{domainId}/appointment/{customerId}`; if they want to buy, give `…/portal/{domainId}/payment/{customerId}`." The server extracts URLs and returns them as clickable links.
   - **No email yet (anonymous):** a different system prompt: "You're an experienced sales rep for {domain}; welcome them warmly and lead the conversation naturally toward getting their email. Never break character."
4. **The owner's inbox** lists chat rooms per domain (latest message, an "urgent" icon if the visitor wrote within the last 2 hours and the message is unseen). Opening a room marks messages seen. Toggling the switch in the top bar sets `live`.
5. **Real-time** used a Pusher channel per chat room; both the widget and the dashboard subscribe.

---

## 5. Chapter-by-chapter

| Time | Chapter | What actually happens |
|---|---|---|
| 0:00–13:14 | Intro and demo | Full product tour (section 1 above). |
| 14:43 | Next.js setup | `bunx create-next-app`; folders `actions, constants, context, hooks, icons, lib, schemas`; `.env` copied from the repo. |
| 17:08 | Clerk auth | Create the Clerk app (email only, Google off); middleware with public routes `/`, `/auth(.*)`, `/portal(.*)`, `/images(.*)`; ignored route `/chatbot`; ClerkProvider. |
| 20:33–25:00 | Prisma and Neon | Prisma client singleton, Neon connection string, schema pasted from GitHub, `prisma generate`, `db push`, Prisma Studio. |
| 26:04–1:23:00 | Custom Clerk auth UI | Auth layout (redirect signed-in users, left form and right marketing image); auth-step context; `useSignUpForm` hook (zod schema, `signUp.create`, `prepareEmailAddressVerification`, `attemptEmailAddressVerification`, `onCompleteUserRegistration` server action that creates the DB user with the STANDARD plan); type-selection cards; FormGenerator (input/select/textarea driven by constants); OTP input; highlight bar; sign-in form and hook. Bug hit: no `<Toaster/>`, so errors were invisible. |
| 1:23:02–1:35:22 | Figma → landing | Navbar generated from Figma via Builder.io; font switched to Plus Jakarta Sans; shadcn theme provider (dark mode); hero, pricing cards from `constants/landing-page`. |
| 1:35:22–2:20:10 | Cloudways headless WordPress blog | Spin up WordPress on Cloudways, install the REST API and Custom Post Type UI plugins, create "test posts", test endpoints in Postman, fetch posts and featured media with axios in a server action, render cards and a `/blogs/[id]` page with `html-react-parser`; `next.config` remote image patterns. Plus a long SEO talk. |
| 2:20:10–2:29:38 | Dashboard routing, infobar, billing card | `(dashboard)` route group; infobar with breadcrumb (page title and description), trash/star icons, avatar, headphones; billing settings server action to read the plan. |
| 2:29:38–2:48:29 | Responsive sidebar | Owner layout fetches the user and domains; chat context (realtime, chatRoom, chats, loading); `useSideBar` hook (expand, real-time toggle, current page from the path, sign out); max and min menus; sidebar menu constants and custom SVG icons. |
| 2:48:29–3:25:02 | Uploadcare and adding domains | `useDomain` hook: zod schema (domain regex, image ≤2 MB png/jpg/jpeg), upload to Uploadcare, `onIntegrateDomain` server action with **plan limits** (Standard 1); bottom drawer form; upload button; domain links with icons. |
| 3:25:02–3:48:30 | Theme and password settings | Theme picker cards (system/light/dark); change-password hook and server action (Clerk `updateUser`); billing card lists plan features. |
| 3:48:30–4:08:31 | Domain settings page | `/settings/[domain]`; `onGetCurrentDomainInfo`; settings form (update domain name, update bot icon, update welcome message, delete domain); **code snippet** component with copy and toast. |
| 4:08:31–4:20:40 | Bot customisation | Pro/Ultimate gate; edit chatbot icon; greeting message; dynamic import for performance; delete button. |
| 4:20:40–4:54:22 | Bot training | Tabs menu; Help Desk (create Q&A and list them in an accordion); Filter Questions (create and list). Server actions per operation. |
| 4:54:22–5:30:07 | Conversations page | `useConversation` hook (domain select with `watch`, load chat rooms), chat cards with relative time and an urgent badge, `useChatWindow` (scroll to bottom, send owner message), the Pusher subscription stubbed out for now, messenger UI with bubbles. |
| 5:30:07–5:59:08 | Chatbot UI and embedding | `/chatbot` page and `AiChatbot` component; test by pasting the snippet into a plain `index.html` served with Live Server; `useChatBot` hook (postMessage handshake, iframe resize, load bot, send message, image upload); bot window (header, real-time badge, chat/help-desk tabs, input, paperclip, "powered by"). Bug: a trailing slash in the iframe URL broke embedding. |
| 5:59:08–6:07:39 | Nodemailer | Gmail SMTP; 2-step verification and an app password; mail the owner when a room goes real-time. |
| 6:07:39–6:45:05 | OpenAI and prompts | The two system prompts and the keyword protocol (section 4); `gpt-3.5-turbo`; getting an API key; live testing (it correctly withholds answers until it has the email, and sends links on intent). |
| 6:45:05–7:21:46 | Customer portal | `/portal/[domainid]/appointment/[customerid]`; questions step (pre-filled, editable), booking-date step (calendar and 30-minute slots, booked slots disabled), thank-you step; `saveAnswers` action; auto-skip step 1 if every question is answered. Bug: markdown links from the AI came out doubled. |
| 7:21:46–7:30:28 | Appointments page | Table of all bookings and the "today" cards (bug: compared timestamps instead of dates). |
| — | **Email marketing** | **Accidentally deleted from the video** (the author posted a Loom instead). Only the demo shows it. This guide designs it from the demo. |
| 7:30:28–7:58:29 | Stripe Connect | Integrations page and modal; `useStripe` → `/api/stripe/connect` route that creates a custom account with **fake business data** to skip onboarding, then an account link. Gotcha: Connect branding (icon) must be set in the Stripe dashboard first. Alternative OAuth method explained. |
| 7:58:29–8:10:50 | Products | Products table per domain, side sheet "Add product" form, upload, `onCreateNewDomainProduct`. |
| 8:10:50–8:38:33 | Portal checkout and subscriptions | Payment page: products and total, PaymentIntent on the connected account, `PaymentElement`. Platform plan upgrade: modal, plan cards, PaymentIntent, `onUpdateSubscription` (plan and credits). Bug: "Ultimate" vs "Unlimited" naming mismatch. |
| 8:38:33–8:56:27 | Dashboard | Metrics server actions (clients count, Stripe balance, bookings count, plan info, total product prices, Stripe charges); dashboard cards; plan-usage progress bars; transactions list. |
| 8:56:27–9:04:48 | Pusher | Create the app, keys, un-comment the realtime code. Bugs: hard-coded cluster; file reset before upload; **duplicate messages** (an optimistic append plus the realtime echo). The widget-side duplicate was left as a "challenge". |
| 9:04:48–end | Sales advice | Career and sales talk, no code. |

**Skipped as non-technical:** Prodigies University promos (around 1:05:20, 4:17:26), the Cloudways sponsor read (1:35:22), the SEO lecture (about 1:46–1:53), the closing sales framework.

---

## 6. Things the video leaves unfinished (and this guide fills in)

1. **The email marketing build**: deleted from the video. This guide designs it from the demo, sending with Resend.
2. **Conversation tabs** (All / Expired / Starred): left as a challenge.
3. **The widget duplicate-message bug**: left as a challenge. The fix is to dedupe by message ID.
4. **Bot colour themes** (background and text colour): behind the paid community. Easy to add here as a Pro feature.
5. **A single "update all chatbot settings" call**: left as a challenge.
6. **Dark-mode hard-coded colours**: left as a challenge.
7. **Saving changed answers on the portal**: half-fixed. This guide saves them on submit.
8. **Real Stripe Connect onboarding** instead of fake custom-account data: this guide uses Express accounts and Account Links (Stripe test mode has built-in "use test data" shortcuts).
9. **Security**: the original relied on Clerk middleware and server actions. Lovable Cloud needs explicit **row-level security**, and the widget and portal must go through validated backend functions.
