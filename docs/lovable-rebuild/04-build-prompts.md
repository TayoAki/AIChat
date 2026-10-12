# 04 — Stage-by-stage Lovable prompts

Twenty-two stages, in build order. Each stage has:
- **Goal**: what you'll have when it's done.
- **Mode**: Build, or **Plan → Approve** for big or risky stages.
- **Prompt**: copy the whole code block into Lovable.
- **Done when**: test these yourself before moving on, then **bookmark the version** in history.

Rules that keep credits low and quality high:
- Send **one stage per message**. If a stage feels too big, split it at the bullet points.
- Don't start the next stage until "Done when" passes. Use the debugging prompts in [`05-testing-and-debugging.md`](05-testing-and-debugging.md).
- Paste [Project Knowledge](03-project-knowledge.md) right after Stage 1. Every prompt below assumes Lovable has it.
- If Lovable asks a clarifying question, answer it. Don't just resend the prompt.
- **Republish before testing.** Lovable doesn't push new code to your published site automatically. From Stage 9 on, many checks run on the published URL (`APP_URL`): the embed test page and widget, the bot's portal links, Stripe redirects, unsubscribe links and the blog's page source. So after each stage click **Publish → Publish changes**, then test. (The database is shared, so data you create in the preview is there too.)

| # | Stage | Video equivalent | Mode |
|---|---|---|---|
| 0 | Accounts and setup | 14:43 | manual |
| 1 | Landing page and theme | 1:23–1:35 | Build |
| 2 | Database schema, RLS, storage | 20:33–25:00 | **Plan → Approve** |
| 3 | Custom 3-step sign-up with OTP, sign-in | 26:04–1:23 | Build |
| 4 | Dashboard shell: sidebar and infobar | 2:20–2:48 | Build |
| 5 | Domains: add drawer, list, plan limits | 2:48–3:25 | Build |
| 6 | Settings: plan card, theme, password | 3:25–3:48 | Build |
| 7 | Domain settings: rename, embed snippet, bot look, delete | 3:48–4:20 | Build |
| 8 | Bot training: Help Desk and filter questions | 4:20–4:54 | Build |
| 9 | Chatbot widget UI, embed script, iframe test | 5:30–5:59 | Build |
| 10 | AI brain edge function (Lovable AI) and handoff email | 5:59–6:45 | **Plan → Approve** |
| 11 | Conversations inbox, realtime, image uploads | 4:54–5:30, 8:56–9:04 | **Plan → Approve** |
| 12 | Customer portal: booking flow | 6:45–7:21 | Build |
| 13 | Appointments page | 7:21–7:30 | Build |
| 14 | Stripe Connect integration | 7:30–7:58 | **Plan → Approve** |
| 15 | Products | 7:58–8:10 | Build |
| 16 | Customer portal: payment flow | 8:10–8:23 | **Plan → Approve** |
| 17 | Platform plan subscriptions | 8:23–8:38 | **Plan → Approve** |
| 18 | Email marketing (Resend) | demo 11:14 (cut from video) | **Plan → Approve** |
| 19 | Dashboard metrics | 8:38–8:56 | Build |
| 20 | SEO blog (built-in CMS or WordPress) | 1:35–2:20 | Build |
| 21 | Hardening, SEO, publish, custom domain | — | Build and manual |

---

## Stage 0 — Accounts and setup (manual, no prompt)

1. Sign up at lovable.dev. **Pro plan** is recommended: you need it for custom domains, Lovable Emails and more credits.
2. Have these ready (you'll paste them when Lovable asks):
   - **Stripe**: an account in **test mode**. In Dashboard → Settings → **Connect**, enable Connect (platform: "Platform or marketplace") and fill in **Connect branding** (business name, icon, colour). Create a **restricted key** (or use the test secret key).
   - **Resend**: an account; add and verify your sending domain (DNS records at your registrar); create an API key (`re_…`, sending access is enough).
3. Optional: decide on a custom domain such as `app.yourbrand.com`.
4. In Workspace settings → Knowledge (optional), add personal rules such as "Always explain what you changed in 3 bullets at the end."

---

## Stage 1 — Landing page and theme

**Goal:** a project exists, with a polished public homepage, pricing and dark mode.
**Mode:** Build (this first prompt creates the project).

```text
Build the public marketing site for "AIChat" — an AI sales-representative chatbot that businesses embed on their own website with one line of code. It captures leads, asks qualifying questions, books appointments and takes payments.

Visual style: minimal, premium, friendly. Font: Plus Jakarta Sans (Google Fonts). Accent: orange (#F97316 family). Light mode background: warm cream (#FDF8F3), cards white. Dark mode: near-black background, zinc cards. Add a theme provider with light / dark / system support and define ALL colors as semantic design tokens (background, foreground, card, muted, primary = orange, border, etc.) so every component works in both themes.

Create src/constants/plans.ts as the single source of truth for plans:
- id "standard", name "Standard", price "Free", description "Perfect if you're just getting started", limits { domains: 1, contacts: 10, emailCredits: 10 }, customization: false
- id "pro", name "Pro", price "$15", description "For growing businesses that want more", limits { domains: 5, contacts: 50, emailCredits: 50 }, customization: true
- id "ultimate", name "Ultimate", price "$35", description "Everything, unlimited domains", limits { domains: null (unlimited), contacts: 500, emailCredits: 500 }, customization: true
Each plan also has a human-readable features[] list derived from its limits (e.g. "1 domain", "10 contacts", "10 emails per month", "Custom chatbot icon & greeting").

Homepage "/" sections:
1. Sticky navbar: logo (small orange bot icon + "AIChat"), links Home, Pricing, News Room, Features, Contact (smooth-scroll anchors, each with a real target: Home → top, Pricing → #pricing, News Room → #news, Features → #features = a section right after the hero with 6 icon cards: lead capture, qualifying questions, appointment booking, payments, live chat takeover, email marketing; Contact → #contact = the footer, which shows a contact email), right side "Free Trial" orange button → /auth/sign-up. On mobile, links go into a slide-out sheet.
2. Hero: small orange pill badge "An AI powered sales assistant chatbot", large headline "Your AI powered sales assistant", sub text "Embed AIChat into any website with just a snippet of code!", primary button "Start For Free" → /auth/sign-up, and an illustration of a phone showing a chat widget conversation (generate it).
3. Pricing (id="pricing"): heading "Choose what fits you right", short paragraph, 3 cards mapped from plans.ts. Card: name, description, price + "/month" (except Free), feature list with check icons, "Get started" button → /auth/sign-up. Ultimate card gets an orange border and a "Most popular" badge.
4. News Room (id="news"): heading "News Room", sub text "Ideas and guides on using AI to grow your business.", a responsive grid of 3 placeholder blog cards (cover image, date, title, 2-line excerpt). Real posts come later.
5. Footer: logo, copyright, links.

Only the frontend for now — no auth, no database yet. Make it fully responsive and use real, specific copy (no lorem ipsum).
```

**Done when:**
- The homepage looks good on mobile and desktop.
- The theme toggle works and nothing becomes unreadable in dark mode.
- Pricing comes from `plans.ts`.

➡️ Now paste [Project Knowledge](03-project-knowledge.md) into **Project settings → Knowledge**. Check that **Cloud** is enabled for the project and choose your region (it's permanent).

---

## Stage 2 — Database schema, RLS, storage

**Goal:** the whole data model exists up front, so later stages don't fight migrations.
**Mode:** **Plan → review → Approve.** Read the plan and check every table and policy.
**Before you start:** a workspace owner/admin must open Workspace settings → Security → **Privacy & security** (Data protection) and turn **off** "Block public storage buckets". It's on by default on every plan and forces every new bucket to private, so the public image URLs used by the sidebar, widget, portal and blog would not load. (If the buckets were already created as private, ask Lovable to make them public.)

```text
Set up the complete Lovable Cloud backend for AIChat using the data model in Project Knowledge. Don't build UI in this step.

1. Enum plan_type ('standard','pro','ultimate'); enum app_role ('admin','user'); enum message_role ('user','assistant','owner'); enum account_type ('owner','student').

2. Create every table listed in Project Knowledge with sensible types, NOT NULL where obvious, foreign keys with ON DELETE CASCADE from children to domains / customers / chat_rooms / campaigns / filter_questions (so deleting a filter question also deletes its customer_responses instead of failing), unique(customer_id, filter_question_id) on customer_responses and unique(campaign_id, customer_id) on campaign_customers (no duplicate rows, double emails or double credit charges), and these indexes: domains(user_id), chat_rooms(domain_id, last_message_at desc), chat_messages(room_id, created_at), customers(domain_id), bookings(domain_id, date), products(domain_id), payments(domain_id, created_at).

3. Trigger on new auth user: insert profiles (full_name and account_type from signup metadata), subscriptions (plan 'standard', credits 10) and user_roles ('user').

4. Trigger on new domain: auto-create its chatbots row with welcome_message "Hey there, have a question? Text us here", show_help_desk true.

5. Plan limit trigger BEFORE INSERT on domains: count the user's domains and raise an exception with message "You've reached the maximum number of domains for your plan. Upgrade to add more." if standard >= 1 or pro >= 5 (ultimate unlimited).

6. Security-definer helper functions (to avoid RLS recursion): has_role(user_id, role), owns_domain(domain_id) → boolean (domains.user_id = auth.uid()), owns_room(room_id), owns_customer(customer_id).

7. Row Level Security ON for every table:
   - profiles / subscriptions: user can select only their own row. profiles: the user may update only full_name and account_type on their own row (REVOKE UPDATE ON profiles FROM authenticated, then GRANT UPDATE (full_name, account_type) ON profiles TO authenticated — or a BEFORE UPDATE trigger that rejects changes to any other column unless the caller is the service role). stripe_account_id and stripe_charges_enabled are written only by edge functions with the service role. subscriptions: select only — plan/credits are changed only by edge functions with the service role.
   - domains: full CRUD only where user_id = auth.uid().
   - chatbots, help_desk, filter_questions, customers, products, bookings, payments, chat_rooms: CRUD only where owns_domain(domain_id).
   - customer_responses: via owns_customer(customer_id). chat_messages: select/insert/update via owns_room(room_id); owners may only insert role 'owner'.
   - campaigns / campaign_customers: owner only (campaigns.user_id = auth.uid()).
   - blog_posts: anyone (anon) can select where published = true; admins (has_role(auth.uid(),'admin')) can also select ALL posts including drafts; only admins can insert/update/delete.
   - user_roles: users can read their own roles; no client writes.
   - Anonymous visitors get NO direct table access (the widget/portal will use edge functions later).

8. Storage buckets: domain-icons, bot-icons, product-images, blog-images (public read; authenticated users may upload only into a folder named after their user id), chat-uploads (public read by file URL; authenticated owners may upload only under {domainId}/{roomId}/… for a domain they own, i.e. owns_domain(((storage.foldername(name))[1])::uuid), because the inbox composer uploads directly; anonymous visitors write only via edge functions). Limit 2 MB and image/png, image/jpeg, image/webp.

9. Enable Realtime for chat_messages and chat_rooms.

Show me the plan with the SQL for review before running it.
```

**Done when:**
- Cloud → Database lists all the tables.
- Cloud → Database → RLS policies shows policies on every table.
- Storage shows 5 buckets.
- No warnings in the Security view, or you've asked Lovable to fix them.

---

## Stage 3 — Custom 3-step sign-up with OTP, and sign-in

**Goal:** branded auth like the video (account type → details → 6-digit code).
**Mode:** Build. First, in Cloud → Users → Auth settings, set **email OTP length 6**, turn **"Auto-confirm email" OFF**, minimum password length 8, and enable the leaked-password check.

```text
Build custom authentication pages with Lovable Cloud Auth (no third-party auth widgets).

Shared auth layout for /auth/*: two columns on desktop. Left: logo + the form (max-w-md). Right (hidden on mobile): muted panel with heading "Hi, I'm your AI powered sales assistant, AIChat!", text "AIChat captures lead information without a single form.", and an app screenshot illustration. If a user is already signed in, redirect from /auth/* to /dashboard, EXCEPT /auth/reset-password: the reset email (resetPasswordForEmail with redirectTo {window.location.origin}/auth/reset-password) signs the user in with a recovery session, and they must stay on that page to set the new password with updateUser({ password }).

/auth/sign-up — a 3-step wizard with an orange 3-segment progress bar at the bottom (current/done segments orange, others muted):
Step 1 "Create an account" — "Tell us about yourself! What do you do? Let's tailor your experience so it best suits you." Two selectable cards (custom radio cards with icon, title, description; selected = orange border + orange filled dot): "I own a business — Setting up my account for my company" (value owner, default) and "I am a student — Looking to learn about the tool" (value student). Button "Continue". Link "Already have an account? Sign In".
Step 2 "Account details" — "Enter your email and password". Fields: Full name, Email, Confirm email, Password, Confirm password. zod: name required, valid email, emails must match, password 8–64 chars letters and numbers, passwords must match; show inline errors. Button "Continue" (disabled until valid) → call signUp with email/password and user metadata { full_name, account_type }. Show a toast on error (e.g. "Password has been found in a data breach" → readable message). If signUp returns no error but data.user.identities is an empty array, the email is already registered: toast "An account with this email already exists — please sign in" and do not go to the OTP step.
Step 3 "Enter OTP" — "Enter the one-time password that was sent to your email." 6-slot OTP input (shadcn InputOTP). Button "Create account" → verifyOtp({ email, token, type: 'signup' }). On success go to /dashboard. Add "Resend code" (cooldown 60s).
Keep all wizard state in one form (react-hook-form) + a small step context so Back keeps the values.

/auth/sign-in — heading "Login" with plain email + password (no one-time code on sign-in), validated with zod (valid email, password 8–64). If sign-in fails with "Email not confirmed" (error code email_not_confirmed), call supabase.auth.resend({ type: 'signup', email }) and show the same 6-digit OTP input (verifyOtp type 'signup') so the user can finish verifying. Button "Submit" with spinner. "Forgot password?" → send reset email → /auth/reset-password page to set a new one. Link "Don't have an account? Create one".

Make sure a <Toaster /> is mounted at the app root. Protect all owner routes listed in Project Knowledge: unauthenticated users are redirected to /auth/sign-in. Update the navbar "Free Trial" button to go to /dashboard if logged in.
```

**Done when:**
- A new email gets a 6-digit code, the account is created, and you land on `/dashboard` (a 404 or placeholder is fine for now).
- In Cloud, the `profiles`, `subscriptions` and `user_roles` rows exist.
- Sign out and sign back in works.

> If the confirmation email arrives as a **link** instead of a code: in Cloud → Emails, edit the signup template to show the token (`{{ .Token }}`), or ask Lovable: *"Change the signup confirmation email template to show the 6-digit code instead of a link."*

---

## Stage 4 — Dashboard shell: sidebar and infobar

**Goal:** the owner layout every dashboard page uses.
**Mode:** Build.

```text
Create the authenticated owner layout used by /dashboard, /conversation, /integration, /settings, /settings/:domainName, /appointment and /email-marketing (create simple placeholder pages for each).

Sidebar (left, full height, muted background):
- Expanded (w-64) and collapsed (w-16, icons only) states with a smooth width transition; a menu icon toggles it; remember the state in localStorage. On mobile it starts collapsed.
- Top: logo (full when expanded, icon when collapsed).
- "MENU" group: Dashboard (layout icon), Conversations (message-square), Integrations (plug), Settings (settings), Appointments (calendar), Email Marketing (mail). Active route highlighted (white/card background, bold). Define items in src/constants/menu.ts.
- "DOMAINS" group: heading with a "+" button (opens an add-domain drawer in a later stage — just a placeholder button now) and a list area for domains (empty for now).
- "OPTIONS" group at the bottom: Sign out (log-out icon) and Mobile App (smartphone icon, no-op).

Infobar (top of the content area):
- Left: page title + one-line description derived from the current route: Dashboard "A detailed overview of your metrics, usage, customers and more"; Conversations "Chat with your customers in real time"; Integrations "Connect third-party applications into AIChat"; Settings "Manage your account settings, preferences and integrations"; Appointments "View and edit all your appointments"; Email Marketing "Send bulk emails to your customers". Put these in src/constants/pages.ts.
- Right: a rounded card with ghost icon buttons (trash, star) — placeholders for now — the user's avatar (initials fallback) and a headphones "support" icon.
- Leave a slot next to the title where a "Real-time" switch will appear later on the Conversations page.

Main content scrolls independently; the whole layout is responsive and uses semantic tokens only.
```

**Done when:** every menu item navigates, the active item highlights, the sidebar collapses and expands, and the infobar title changes per page in both themes.

---

## Stage 5 — Domains: add drawer, list, plan limits

**Goal:** owners can add domains (one chatbot each); Standard is limited to one.
**Mode:** Build.

```text
Implement domains.

1. Hook src/hooks/use-domains.ts: loads the current user's domains (id, name, icon_url) ordered by created_at, and exposes addDomain({ name, iconFile }).
2. The "+" in the sidebar DOMAINS group opens a bottom Drawer (shadcn Drawer) titled "Add your business domain" with description "Add in your domain address to integrate your chatbot". Form (react-hook-form + zod):
   - Domain: text input, placeholder "mydomain.com", min 4 chars, must match a domain regex (letters/numbers/hyphens with a TLD, no protocol, no slashes) → error "This is not a valid domain".
   - Icon: custom upload button (hidden file input + label with an edit/upload icon, text "Upload Icon", hint "Recommended size 300px * 300px, less than 2MB"), required, png/jpg/jpeg/webp only (the bucket rejects other types such as GIF/SVG), max 2 MB, inline zod errors, show a preview.
   - Button "Add Domain" with a loading spinner.
   On submit: upload the icon to the domain-icons bucket at {userId}/{uuid}.{ext}, get the public URL, insert the domain. If the database plan-limit trigger raises, show its message in a destructive toast; otherwise toast "Success — Domain successfully added", close the drawer, reset the form AFTER the upload finishes, and refresh the list.
3. Sidebar DOMAINS list: each domain links to /settings/{domain name} and shows its icon (rounded) + name (name hidden when collapsed); highlight the active one. Empty state: small muted text "No domains yet".
4. Also refuse duplicates with a friendly toast "Domain already exists".
```

**Done when:**
- Adding a domain works and it appears in the sidebar with its icon.
- On Standard, a second domain shows the plan-limit toast.
- An invalid domain or an oversized image shows inline errors.

---

## Stage 6 — Settings: plan card, theme, password

**Goal:** the `/settings` page from the demo (the upgrade button gets wired in Stage 17).
**Mode:** Build.

```text
Build /settings with three sections, each laid out as a 2-column grid on desktop (left: section label + description, right: content), separated by dividers:

1. "Billing settings" — "Add payment information, upgrade and modify your plan." Right side:
   - If plan is standard: a dashed-border clickable card "+ Upgrade" / "Upgrade your plan" (opens nothing yet — TODO Stage 17).
   - Otherwise: a nice credit-card style gradient card showing the plan name.
   - Below: "Current Plan" — plan name (from subscriptions) + its description, and the feature list from src/constants/plans.ts with check-circle icons (muted color). Never compare plan ids with display names; use the enum id.
2. "Interface Theme" — "Select or customize your UI theme". Three clickable preview cards (System, Light, Dark) drawn as small mock UI illustrations; the active one has an orange border; clicking sets the theme.
3. "Change Password" — "Reset your password". Fields New password + Confirm password (zod: 8–64 chars, letters/numbers, must match → "Your passwords do not match"). Button "Change Password" with spinner → supabase.auth.updateUser({ password }). Toast "Success — Password updated", reset form.
```

**Done when:** the theme cards switch the theme, the password change works (sign in again with the new password), and the plan shows "Standard" with its features.

---

## Stage 7 — Domain settings: rename, embed snippet, bot look, delete

**Goal:** `/settings/:domainName`, the heart of the product.
**Mode:** Build.

```text
Build the domain settings page at /settings/:domainName (look up the domain by name for the current user; if not found or not owned → redirect to /dashboard). Infobar title "Settings".

Section "Domain Settings" (H2) with a separator, then:
1. Domain name: input pre-filled with the current name (same validation as when adding). Saving renames it. Names are unique per owner (unique(user_id, name), stored lowercase), not globally — never add a global unique constraint. Rely on that constraint and turn a duplicate-key error (code 23505) into the toast "Domain already exists". After saving, navigate to the new URL /settings/{newName} and refresh the sidebar.
2. "Code snippet" — "Copy and paste this code snippet into the header tag of your website". Show a read-only code block (cream/muted background, monospace) containing:
   <script src="{APP_ORIGIN}/embed.js" data-bot-id="{domain.id}" async></script>
   where APP_ORIGIN = window.location.origin with NO trailing slash (secrets like APP_URL can't be read in the browser). The editor preview address requires a Lovable login, so a snippet copied there never works on customer sites: when the hostname contains "preview--" or ends with ".lovableproject.com", show a muted warning under the block: "This is a preview address — copy the snippet from your published app." A copy icon button copies it and toasts "Copied to clipboard — You can now paste the code inside your website".
3. "Chatbot Settings" with a "Premium" badge. Only for pro/ultimate plans; for standard show a locked card "Upgrade to Pro to customize your chatbot" with a link to /settings. Enforce it server-side too: a BEFORE UPDATE trigger on chatbots raises "Upgrade to Pro to customize your chatbot" when the domain owner's plan is standard and welcome_message, icon_url, background_color, text_color or show_help_desk changed (IS DISTINCT FROM).
   - "Chatbot icon" — current icon (or a default orange bot icon) + upload button "Edit image" (2 MB, png/jpg/webp) → bot-icons bucket at {userId}/{uuid}.{ext} (storage policies only allow the user's own folder) → public URL → chatbots.icon_url.
   - "Greeting message" — "Customize your welcome message", textarea (2 lines min), pre-filled from chatbots.welcome_message.
   - "Bot colors" — two color pickers: window background and text color (stored on chatbots), with a small live preview bubble.
   - "Show Help Desk tab" switch (chatbots.show_help_desk).
   - On the right side (desktop only, md:grid-cols-2): an illustration of the chat widget.
4. One "Save" button saves ALL changed fields in a single update (only upload a new image if one was chosen), shows a spinner and a toast.
5. "Delete Domain" destructive button with a confirm AlertDialog ("This deletes the chatbot, its conversations, customers, questions and products.") → delete → toast → /dashboard, refresh sidebar.

Leave room below for a "Bot Training" section (Stage 8) and a "Products" section (Stage 15).
```

**Done when:**
- Rename, colour, greeting, icon and delete all persist. Everyone is on Standard until Stage 17, so to test the Premium fields temporarily run `update subscriptions set plan = 'pro' where user_id = '<your user id>';` in Cloud → Database → SQL editor (your id is in Cloud → Users; set it back to `standard` afterwards).
- The snippet has no trailing slash. In the editor preview it shows the preview warning; on the published app it uses your real origin (always copy it from there).
- A Standard user sees the locked card.

---

## Stage 8 — Bot training: Help Desk and filter questions

**Goal:** owners teach the bot FAQs and the qualifying questions it must ask.
**Mode:** Build.

```text
On /settings/:domainName add a "Bot Training" section below Chatbot Settings: heading "Bot Training", text "Set FAQ questions, create questions for capturing lead information and train your bot to act the way you want it to." Then shadcn Tabs (styled as a pill tab list) with two tabs:

Tab "Help Desk" (FAQ shown to visitors in the widget and used by the AI to answer):
- Left card "Help Desk": form with "Question — Add a question that you believe is frequently asked." (input) and "Answer to question — The answer for the question above." (textarea, 5 lines) and a "Create" button. zod: both required.
- Right card: list of existing FAQs as an Accordion (question = trigger, answer = content, preserve line breaks). Each item has a delete (trash) button with confirm. Empty state "No Questions".

Tab "Questions" (filter questions the AI must get answered to qualify leads):
- Left card "Question": "Add a question that you want your chatbot to ask" (input) + "Create" button. New questions get sort_order = current max + 1 (not the list length, which collides after a delete); reordering renumbers sort_order so every question keeps a distinct value.
- Right card: ordered list of questions with drag handles or up/down arrows to reorder (sort_order), and delete. Empty state "No Questions". Delete asks for confirmation ("This also deletes the answers customers gave to it.") and must keep working after customers have answered: customer_responses.filter_question_id must reference filter_questions ON DELETE CASCADE (add a migration if it doesn't).
- Small helper text: "Tip: ask qualifying questions like 'What's your budget?', 'Are you ready to start this month?'"

Use one hook per tab (use-help-desk.ts, use-filter-questions.ts) that loads on mount and updates the list optimistically after create/delete, with toasts. On mobile the two cards stack.
```

**Done when:** you can add, list and delete FAQs and questions, reorder the questions, and everything survives a refresh.

---

## Stage 9 — Chatbot widget UI, embed script, iframe test

**Goal:** the widget renders inside an iframe on any website, with the welcome message and Help Desk working. No AI yet.
**Mode:** Build.

```text
Build the embeddable chatbot.

A) Public edge function "chatbot-config" (no auth required, CORS open): input { domainId }. Returns { domainName, chatbot: { welcome_message, icon_url, background_color, text_color, show_help_desk }, helpDesk: [{question, answer}] } for that domain or 404. Uses the service role; never returns owner data.

B) Route /chatbot?id=DOMAIN_ID — a standalone page with NO app layout, transparent page background (html/body background transparent), no auth. Always render this route in the light theme (ignore the app theme provider and the visitor's system dark mode) with color-scheme: normal on its <html>. Otherwise browsers paint an opaque box behind the transparent iframe on the host site when its color scheme differs from the iframe element's. It renders only the widget:
- Closed state: a 64px round floating button bottom-right showing the bot icon (or default orange bot icon). Click → open.
- Open state: the window fills the whole iframe (w-full h-full). embed.js decides the size (see the resize rule below and C), so there is no fixed 380×620 size and no media query inside the widget. Rounded corners, border and shadow, colored with the chatbot's background/text colors when set:
  • Header: bot avatar with a green online dot, title "Sales Rep — {domainName without TLD}", subtitle "{domainName}", a close (chevron-down) button, and a "Real time" badge area (hidden for now).
  • Tabs: "Chat" and "Help Desk" (Help Desk tab only if show_help_desk).
  • Chat tab: scrollable message list (auto-scroll to bottom on new messages) starting with the welcome message as an assistant bubble. Bubbles: assistant left (muted, with bot avatar), user right (orange, with user icon), timestamp under each. URLs inside messages render as clickable links that open in a new tab (target _blank, rel noopener). Images render inline.
  • Composer: input "Type your message...", a paperclip button (file input, images only, 2 MB — wiring later), a send button. Enter sends. For now, sending just appends the user bubble locally and shows a "typing…" three-dot indicator for 1 second, then a placeholder assistant reply "(AI coming soon)".
  • Help Desk tab: "Help Desk — Browse from a list of questions people usually ask." + accordion of FAQs.
  • Footer: small muted text "Powered By AIChat".
- Whenever the widget opens or closes, post a message to the parent window: window.parent.postMessage({ type: "aichat:resize", open: true|false }, "*"). Never decide "mobile" or sizes inside the widget: its media queries and useIsMobile only see the iframe (80px wide when closed), not the visitor's screen. embed.js sizes the iframe: closed 80×80 at bottom/right 16px; open 380 × min(620, host window innerHeight − 32) px; full-screen (top/left/right/bottom 0, width/height 100%) when the host window is narrower than 480px. Re-apply on host window resize and restore the corner position on close. The open window always fills the iframe (w-full h-full).
- Show a friendly error card if the id is missing/invalid.

C) Static file public/embed.js (plain vanilla JS, no framework, no build step): reads data-bot-id from its own <script> tag (document.currentScript), derives the app origin from the script src, creates an <iframe> pointing to `${origin}/chatbot?id=${botId}` with style position:fixed; bottom:16px; right:16px; width:80px; height:80px; border:0; background:transparent; z-index:2147483647; color-scheme normal; allow="clipboard-write". Listen to window "message" events; only accept events whose event.origin === origin and data.type === "aichat:resize", then set the iframe width/height (with a short transition). Guard against loading twice. Read document.currentScript synchronously at the top of the script (it's null inside callbacks). The snippet goes in <head> with async, so document.body may not exist yet when it runs: if so, wait for DOMContentLoaded before appending the iframe.

D) Create public/embed-test.html: a plain fake business website (heading, some lorem text) that includes <script src="/embed.js" data-bot-id="PASTE_DOMAIN_ID" async></script>, so I can test quickly.
```

**Done when:**
- `/chatbot?id=<your domain id>` shows the bubble, opens and closes, and shows the Help Desk FAQs.
- **Publish the app**, put the snippet from `/settings/:domain` into a plain HTML file **on another origin** (open it locally, or a CodePen/JSFiddle page), and confirm the bubble appears and resizes.
- ⚠️ If the iframe is blocked (the browser console shows an `X-Frame-Options` or `frame-ancestors` error), switch to Chat mode and ask: *"The /chatbot route must be embeddable in iframes on any website. How do I allow framing for that route only on Lovable hosting?"*. If there's no option, contact Lovable support before going further. Everything else still works on the hosted page itself.

---

## Stage 10 — AI brain edge function (Lovable AI) and handoff email

**Before you start:** connect **Resend** (Connectors → Resend → paste the key and link it to this project), and add secrets `FROM_EMAIL` (for example `AIChat <notify@yourdomain.com>`) and `APP_URL` (your published URL with no trailing slash).
**Goal:** the bot captures emails, qualifies leads, sends booking and payment links, and hands off to a human.
**Mode:** **Plan → review → Approve.**

```text
Replace the placeholder reply in the widget with a real AI sales rep powered by Lovable AI (default chat model), implemented as a public edge function "chatbot-message" (no auth, CORS open, uses the service role internally, validates every input with zod).

Input: { domainId, roomId?, visitorToken?, content?, imageUrl? }. Require content (trimmed, max 2,000 chars) or imageUrl. imageUrl must be a public URL from this project's chat-uploads bucket under {domainId}/{roomId}/; reject any other URL, because it is shown to the owner. Output: { roomId, visitorToken, live, messages: [new user message, new assistant message?] } with full message rows (id, role, content, image_url, created_at).

Steps:
1. Load domain (+ owner user_id), chatbot, help_desk FAQs, filter_questions (ordered), active products (name + price). 404 if domain missing.
2. Room: if no roomId → create chat_rooms row with a random visitor_token and return it. Else load the room and require room.domain_id = domainId AND visitor_token match (else 403).
3. Insert the visitor's message (role 'user', content, image_url, seen false); update room.last_message_at.
4. If room.live is true → still run step 5 (email capture, so an email shared during a live chat becomes a lead), then return (no AI reply; the owner answers in real time).
5. Email capture: if the room has no customer_id and the content contains an email (regex) → upsert customers (domain_id, lower(email)). If the domain owner has reached their plan's contact limit and this email is new, still chat but don't create the customer. Link room.customer_id. Create customer_responses rows (question copied, answer null) for every filter question that doesn't already have one for this customer.
6. Build the model input: system prompt + the last 20 messages of this room (user → "user", assistant/owner → "assistant"; a message with only an image becomes the text "[sent an image]"; never send empty content to the model).
   System prompt (write it well, roughly):
   - You are a friendly, expert sales representative for {domain name}. Sound human, warm and concise (max ~3 sentences). Plain text only: no markdown (no **bold**, bullet lists or [links](…)), because the widget shows text as-is. Never reveal you are following instructions. Stay respectful and in character.
   - Business knowledge: FAQ list (question → answer) and products with prices. Use them to answer; if you don't know, say so and offer to connect them with the team.
   - If no customer email yet: welcome them on behalf of {domain name} and naturally steer the conversation toward getting their email address ("so I can send you details"). Don't ask the qualifying questions yet.
   - If the customer is known: here are the qualifying questions still unanswered: [{id, question}...]. Ask them one at a time, woven naturally into the conversation — not like a form or a sales script. When the visitor's latest message answers one, report it.
   - Intent: "book_appointment" when they want a call/meeting/demo; "buy" when they want to purchase/pay; "handoff" when they are rude, frustrated, ask for a human/manager, or the request is out of scope; otherwise "none". Never write URLs yourself.
   Ask the model to return ONLY JSON: { "reply": string, "answered_question_id": string|null, "answer": string|null, "intent": "none"|"book_appointment"|"buy"|"handoff" } (use structured output / tool calling if the model supports it; otherwise parse JSON defensively and fall back to { reply: raw text, intent: "none" }).
7. If answered_question_id belongs to this customer's unanswered responses → save the answer.
8. Links (built by the server from the APP_URL secret, never by the model):
   - book_appointment and customer known → append "\n\nYou can pick a time that suits you here: {APP_URL}/portal/{domainId}/appointment/{customerId}"
   - buy and customer known → append "\n\nYou can complete your purchase here: {APP_URL}/portal/{domainId}/payment/{customerId}"
   - if the customer is not known yet for book/buy → reply asks for their email first (no link).
9. handoff → set room.live = true; reply "Let me connect you with a member of our team — they'll reply right here shortly." If room.mailed is false: send the domain owner (auth user email) an email via Resend from FROM_EMAIL: subject "A customer on {domain} needs your help", body with the last few messages (HTML-escaped, because visitor text is untrusted) and a button "Open conversation" → {APP_URL}/conversation; then set mailed = true.
10. Insert the assistant message (role 'assistant'), update last_message_at, return.
Handle AI errors: on 429 or 402 reply "Sorry, I'm having trouble right now — please try again in a moment." and log the error.

Widget changes:
- Persist { roomId, visitorToken } in localStorage under key aichat:{domainId}; on load, if present, fetch history through a second public function "chatbot-history" ({ domainId, roomId, visitorToken } → messages + live flag).
- Send messages through chatbot-message; show the typing indicator while waiting; append returned rows (dedupe by id).
- When live is true show the header badge "Real time" (green dot).
```

**Done when** (test on the published embed page):
1. "Hi" → a warm greeting that asks for your email.
2. Give an email → a new **customer** appears in Cloud with `customer_responses` rows.
3. The bot asks your filter questions one at a time; answers get saved.
4. "I'd like to book a call" → a correct appointment link (your real domain, not localhost; a single link, no markdown brackets).
5. "I want to buy" → a payment link.
6. "I want to talk to your manager, you're useless" → the handoff message, `live = true`, and **one** email in your inbox. The bot then stays silent, because live chat arrives in Stage 11. To check the email is sent only once, set that room's `live` back to false in Cloud → Database and repeat it: no second email.

---

## Stage 11 — Conversations inbox, realtime, image uploads

**Goal:** the owner sees every chat and can take over live; both sides update instantly.
**Mode:** **Plan → review → Approve.**

```text
Build /conversation (two panes on desktop, list → detail on mobile) with realtime in both directions.

Left pane:
- Domain select (the owner's domains; default first).
- Tabs: Unread (rooms with unseen visitor messages), All, Expired (no activity for 7+ days), Starred.
- Room list (chat cards), newest activity first: avatar, customer email or "Anonymous visitor", last message preview (first 30 chars + "…", or "This chatroom is empty"), relative time ("2:45 PM" today, "Mon", or "12 Mar"), an orange "urgent" icon when the latest visitor message is unseen and newer than 2 hours, and a live green dot when room.live. Empty state: "No chats for your domain".
- Hook use-conversations.ts loads rooms for the selected domain and subscribes to realtime changes on chat_rooms filtered by domain_id=eq.{domainId} (chat_messages has no domain_id, so it can't be filtered by domain). On each room event, refetch that room's latest message so the list re-sorts and badges update live. Add a SECURITY DEFINER trigger AFTER INSERT on chat_messages that sets chat_rooms.last_message_at = NEW.created_at, so visitor, AI and owner messages all bump the room.

Right pane (messenger):
- Header shows the customer email; the infobar title area shows a "Real-time" Switch for the selected room (bound to chat_rooms.live; toast "Realtime mode enabled/disabled"). Star button in the infobar toggles chat_rooms.starred.
- Message list with the same bubble component as the widget (visitor on the left, AI and owner on the right — owner bubbles orange with "You", AI bubbles muted with a bot icon), images inline, auto-scroll to bottom. When a room is opened, mark its visitor messages seen = true.
- Composer: input + paperclip (image upload to chat-uploads at {domainId}/{roomId}/…) + send. The chat-uploads storage policy must let authenticated owners upload when the first folder is a domain they own (owns_domain((storage.foldername(name))[1]::uuid)), with the same 2 MB and png/jpg/webp limits — add that INSERT policy if it is missing. Owner messages insert into chat_messages with role 'owner'. Disable the composer with a hint "Turn on Real-time to reply" when live is false.
- Empty state: "No chat selected".

Realtime:
- Dashboard: subscribe to postgres changes on chat_messages filtered by room_id (RLS protects it).
- Widget (anonymous): the visitor can't use RLS-protected subscriptions or private channels. Create a SECURITY DEFINER trigger AFTER INSERT on chat_messages that looks up the room's visitor_token and calls realtime.send(to_jsonb(NEW), 'new_message', 'room:' || visitor_token, false). The last argument false makes it a PUBLIC broadcast: realtime.broadcast_changes and private channels need a signed-in user, so with them the anonymous widget would silently get nothing. The widget subscribes to the public channel "room:{visitorToken}" (private: false). The topic is the secret token, not the room id, so nobody else can listen in or inject messages. If broadcast isn't available, fall back to polling "chatbot-history" while the widget is open: every 3 seconds when live, every 10 seconds otherwise, so the widget also notices when the owner turns Real-time on.
- Also broadcast room live changes (SECURITY DEFINER trigger on chat_rooms UPDATE OF live, same public "room:{visitorToken}" topic, event "room_live") so the widget's "Real time" badge appears/disappears instantly. In the widget, show role 'owner' messages on the business side (left, like the bot) with a small "Team" label; only role 'user' bubbles go on the right.
- IMPORTANT: append messages by id with dedupe (Map keyed by id) in BOTH the widget and the dashboard — the sender's own optimistic/returned message and the realtime echo must not show twice. Re-sort by created_at.

Widget uploads: wire the paperclip to a public edge function "chatbot-upload" ({ domainId, roomId, visitorToken, file as base64 }) that validates the token, type (png/jpg/webp) and size (2 MB), stores the image in chat-uploads/{domainId}/{roomId}/{uuid}.ext and returns { imageUrl }. The widget then sends it through chatbot-message with imageUrl (content may be empty) and shows the returned rows. Keep the paperclip disabled until the first text message has created a room. Upload FIRST, then reset the composer.
```

**Done when:**
- Two browser windows (the embed page and `/conversation`): visitor messages appear live in the inbox.
- Toggle real-time → the widget badge shows; the owner's replies appear live in the widget; the AI stays silent.
- Images work both ways. **No duplicate bubbles** on either side.
- The urgent icon and Unread tab behave as expected.

---

## Stage 12 — Customer portal: booking flow

**Goal:** the appointment link from the bot leads to a 3-step booking page.
**Mode:** Build.

```text
Build the public customer portal page /portal/:domainId/appointment/:customerId (no auth; minimal layout: top banner with the AIChat logo, centered content, mobile friendly).

Data comes ONLY through public edge functions that validate the customer belongs to the domain:
- "portal-data" { domainId, customerId } → { domainName, customerEmail, questions: [{ id (the customer_responses row id), question, answer }], bookings: [{ date, slot }] for that domain from today onward, products: the domain's active products [{ id, name, image_url, price_cents }] (used later by the payment page) }. First create any missing customer_responses rows for filter questions the owner added after this customer was captured, so new questions show up too.
- "portal-book" { domainId, customerId, answers: [{ id, answer }], date, slot } → saves answers (customer_responses), inserts bookings (unique constraint prevents double booking → return a friendly error "That slot was just taken"), returns ok.

Steps with a 3-segment orange progress bar:
1. "Account details" — "Please complete the questions below" — one textarea/input per filter question, pre-filled with any answer the bot already captured (editable). All required. "Next". If every question already has an answer, start at step 2 automatically (the visitor can still go back with "Edit Questions").
2. "Book a meeting" — "Discovery call" + a one-line description of what the call covers (e.g. "A 30-minute call to understand your goals and see how we can help.") Left: shadcn Calendar (disable past dates). Send the picked day to portal-book as a plain 'YYYY-MM-DD' string from the visitor's local calendar date (date-fns format(date, 'yyyy-MM-dd')), never date.toISOString(), which shifts the day for visitors east of UTC. Right: grid of 30-minute slot cards from src/constants/time-slots.ts (9:00am–5:30pm). Slots already booked for that date are disabled/greyed; the selected slot is orange. Buttons "Edit Questions" (back) and "Book Now" (submit → portal-book; one save for answers + booking).
3. Thank-you: "Thank you" + "Thank you for taking the time to fill in this form. We look forward to speaking to you soon." showing the booked date/time.
Invalid domain/customer → "This link is invalid or expired."
```

**Done when:**
- The bot's link opens the portal with your answers pre-filled.
- Booking saves a row; that slot then shows disabled; a double booking gets the friendly error.
- Edited answers are saved.

---

## Stage 13 — Appointments page

**Goal:** the owner sees all bookings and today's at a glance.
**Mode:** Build.

```text
Build /appointment for the owner (RLS-protected queries).
Left (2/3 width): table "All appointments" with columns Name (customer email), Requested time (booking date + slot), Created (date the booking was made, with time), Domain. Sorted by upcoming first (by date, then by the slot's order in src/constants/time-slots.ts, not alphabetically, or "10:00am" sorts before "9:00am"), past appointments greyed. Empty state "No appointments".
Right (1/3): "Bookings for Today" — cards for bookings whose DATE equals today's date in the user's timezone (compare dates, not timestamps): slot time, domain, customer email, avatar. Empty state "No appointments for today".
Mobile: stack.
```

**Done when:** today's booking shows on the right, and every booking shows in the table.

---

## Stage 14 — Stripe Connect integration

**Before you start:** Stripe test mode, with **Connect enabled and branding set** (Stage 0). If you made a restricted key, it needs write access to Connect accounts (including account links and login links), Checkout Sessions, Customers, Products/Prices, Subscriptions and the customer portal, or calls fail with permission errors. The `sk_test_` secret key is simpler. Stripe sends you back to `APP_URL` (the published site), so **publish** first and run this stage's test while signed in on the published site: the editor preview is a different origin with its own login.
**Goal:** each business connects its own Stripe account (an Express account) so customers pay them directly.
**Mode:** **Plan → review → Approve.**

```text
Add payments using MY OWN Stripe account (not Lovable built-in payments) — I'll provide a test secret/restricted key; store it as STRIPE_SECRET_KEY. We need Stripe Connect so each AIChat owner gets paid by their own customers.

1. /integration page: grid of integration cards from src/constants/integrations.ts. For now one: Stripe — logo, "Stripe is the fastest and easiest way to integrate payments and financial services into your software platform or marketplace." and a "Connect" / "Connected" button.
2. Clicking opens a Dialog: AIChat logo ⇄ Stripe logo, title "Connect Stripe Account", description "Get paid directly by your customers through Stripe Connect.", a list with check icons ("Accept payments for your products", "Get paid directly to your bank account", "Your data is secure"), buttons "Learn more" (stripe.com/connect, new tab) and "Connect to Stripe" (spinner while loading).
3. Edge function "stripe-connect-onboard" (auth required): if profiles.stripe_account_id is empty → stripe.accounts.create({ type: 'express', email: user email, capabilities: { card_payments: { requested: true }, transfers: { requested: true } }, metadata: { user_id } }) and save the id. Then stripe.accountLinks.create({ account, type: 'account_onboarding', refresh_url: {APP_URL}/integration?stripe=refresh, return_url: {APP_URL}/integration?stripe=return }) → return url; the browser redirects to it. (In test mode Stripe offers "use test data" shortcuts during onboarding.)
4. Edge function "stripe-connect-status" (auth required): retrieves the account, updates profiles.stripe_charges_enabled from account.charges_enabled, returns { connected, chargesEnabled, detailsSubmitted }. Call it when /integration loads and when returning with ?stripe=return.
5. Button states: not connected → "Connect"; onboarding incomplete → "Finish setup" (new account link); charges enabled → "Connected" (disabled) + small "Open Stripe dashboard" link via stripe.accounts.createLoginLink.
Show clear toasts on errors (e.g. if Stripe says Connect branding/icon is missing, show that message).
Security: only these edge functions (service role) write profiles.stripe_account_id and profiles.stripe_charges_enabled. The browser must not be able to change those two columns (profile updates from the client are limited to full_name and account_type) — check this and add a migration if it is not the case.
```

**Done when:**
- Connect → Stripe onboarding (use the test-data shortcuts) → back to `/integration`, which shows **Connected**.
- `profiles.stripe_account_id` is set, and the account appears in Stripe → Connect → Accounts.

---

## Stage 15 — Products

**Goal:** owners add products per domain, to sell on the payment portal.
**Mode:** Build.

```text
On /settings/:domainName add a "Products" section below Bot Training: heading "Products", text "Add products to your store and set them live to accept payments from your customers."
- Tabs: All products, Live, Deactivated (filter by products.active) and on the right an "Add Product" button with a plus icon.
- "Add Product" opens a right-side Sheet "Add a product" — "Add products to your store and set them live to accept payments from your customers." Form: Product name (required), Product image (upload to the product-images bucket at {userId}/{uuid}.{ext}, the owner's own folder that the storage policy requires; 2 MB, png/jpg/webp), Price (in dollars, at least 0.50 because Stripe rejects smaller charges → stored as price_cents = Math.round(price * 100)). New products start active (Live). Button "Create Product" with spinner. Toast and reset after success.
- Table: Featured image (thumbnail), Name, Pricing ($), Created (date), Active switch, delete (confirm).
- If the owner hasn't connected Stripe yet (profiles.stripe_charges_enabled false), show a small warning banner "Connect Stripe in Integrations to accept payments" linking to /integration.
```

**Done when:** you can create a product with an image, toggle it active, and delete it.

---

## Stage 16 — Customer portal: payment flow

**Goal:** the payment link from the bot lets the visitor pay the business via Stripe.
**Mode:** **Plan → review → Approve.**

```text
Build /portal/:domainId/payment/:customerId (same portal layout and portal-data function as the booking page).

Steps with the 3-segment progress bar:
1. Same questions step as the appointment portal (pre-filled, editable, auto-skip if complete). "Next" saves answers via a public edge function "portal-save-answers".
2. "Payment": order summary — every ACTIVE product of the domain as a card (image, name, price) and the Total. Button "Pay {total}" → public edge function "portal-checkout" { domainId, customerId } which:
   - loads the domain owner's profiles.stripe_account_id (error "This business can't accept payments yet" if missing or charges not enabled),
   - sums active product prices server-side (never trust the client),
   - creates a Stripe Checkout Session (mode 'payment', ui_mode 'embedded_page', else hosted; Stripe API version 2026-03-25 renamed 'embedded' to 'embedded_page', so use the value that matches the apiVersion the function pins) with line_items per product (price_data, currency 'usd'), customer_email = customer's email, payment_intent_data: { transfer_data: { destination: stripe_account_id } } plus application_fee_amount only when the fee is above 0 (fee from an optional PLATFORM_FEE_PERCENT secret, default 0), metadata { domain_id, customer_id }, return_url {APP_URL}/portal/{domainId}/payment/{customerId}?session_id={CHECKOUT_SESSION_ID} (embedded sessions take only return_url: Stripe rejects success_url and cancel_url there, so the page's own "Back" button returns to step 2; if hosted, use success_url = that URL and cancel_url = the payment page),
   - inserts a payments row with status 'pending' and the session id, returns clientSecret (embedded) or url (hosted).
   Render Stripe Embedded Checkout inside the page (@stripe/react-stripe-js EmbeddedCheckoutProvider + loadStripe from @stripe/stripe-js, browser only) with my publishable key (pk_test_…), which I'll add as the secret STRIPE_PUBLISHABLE_KEY; portal-checkout returns it with the clientSecret (a pk_ key is public; STRIPE_SECRET_KEY never leaves the function). Redirect instead if hosted.
3. On return with session_id → public edge function "portal-confirm" retrieves the session, verifies payment_status === 'paid' and metadata matches, marks the payment row 'paid' (amount, currency, customer email), then shows step 3: "Thank you" + "Your payment was successful."
Also add a PUBLIC edge function "stripe-webhook" (JWT verification OFF, because Stripe sends no auth header; verify the raw body from req.text() with await stripe.webhooks.constructEventAsync(body, signature, STRIPE_WEBHOOK_SECRET, undefined, Stripe.createSubtleCryptoProvider()), because the sync constructEvent fails in Deno) handling checkout.session.completed to mark payments paid even if the visitor closes the tab — tell me which URL and events to register in the Stripe dashboard.
```

**Done when:**
- The bot's buy link → questions → pay with `4242 4242 4242 4242` → thank you.
- The payment row is `paid`.
- In Stripe test mode, the charge shows a transfer to the connected account.

---

## Stage 17 — Platform plan subscriptions

**Goal:** owners upgrade Standard → Pro/Ultimate; limits and credits update.
**Mode:** **Plan → review → Approve.**

```text
Wire the "Upgrade" card on /settings to real subscriptions using my Stripe account (same STRIPE_SECRET_KEY). Plans and limits come from src/constants/plans.ts.

1. Create (with my approval) two recurring Stripe prices: Pro $15/month and Ultimate $35/month; store the price ids in plans.ts (or secrets STRIPE_PRICE_PRO / STRIPE_PRICE_ULTIMATE).
2. Upgrade modal (Dialog "Choose a plan" — "Tell us about yourself! What do you do? Let's tailor your experience so it best suits you."): three selectable plan cards (radio style, current plan pre-selected and marked "Current"). Selecting Pro/Ultimate shows a "Continue to payment" button.
3. Edge function "create-subscription-checkout" (auth): finds/creates the Stripe customer for the user (save stripe_customer_id), creates a Checkout Session mode 'subscription' with the chosen price, success_url {APP_URL}/settings?upgraded=1, cancel_url {APP_URL}/settings. If the user already has an active subscription, don't create a second one: return a customer-portal session URL instead (plan switches happen there).
4. Edge function "sync-subscription" (auth): looks up the customer's active subscription in Stripe, maps price → plan, and updates subscriptions (plan, stripe_subscription_id, current_period_end). Read the period end from subscription.items.data[0].current_period_end (Stripe API 2025-03-31.basil and later removed subscription.current_period_end) and guard against a missing value so the function never throws "Invalid time value". When the plan changes upward, set credits to the new plan's monthly emailCredits. If no active subscription → plan 'standard'. Call it on /settings load and after returning from checkout; toast "Success — Your plan is now Pro".
5. "Manage subscription" button for paid plans → Stripe customer portal session (cancel/switch plans), then sync on return. The portal must be activated once in the Stripe dashboard (Settings → Billing → Customer portal: allow cancelling and switching between the Pro and Ultimate prices) — tell me exactly what to set. (It doesn't work inside the Lovable preview panel; I'll test it in a separate tab or the published app.)
6. Add the subscription events (customer.subscription.created/updated/deleted, invoice.paid) to the existing stripe-webhook function so plan and credits stay correct even without visiting /settings. Find the user by the event's Stripe customer id (subscriptions.stripe_customer_id) and reuse the sync-subscription logic for that customer; don't read invoice.subscription (newer API versions moved it to invoice.parent.subscription_details.subscription). The existing checkout.session.completed handler must ignore sessions with mode 'subscription' (they have no payments row) and still return 200. Tell me which events to add to the webhook endpoint in the Stripe dashboard. On invoice.paid (renewal) reset credits to the plan's monthly amount.
7. Update the domain-limit trigger and the contact-limit check to read the plan from subscriptions (pro 5 domains, ultimate unlimited).
8. Billing card: once on a paid plan show the gradient card with the plan name and features, and the renewal date.
```

**Done when:**
- Upgrading with the test card switches you to Pro (or Ultimate), and the features and credits update.
- You can add more domains.
- Cancelling in the portal brings you back to Standard after sync or the webhook. (By default the portal cancels at the end of the billing period, so the plan stays Pro until the renewal date; to test the downgrade now, cancel the subscription immediately in the Stripe dashboard.)

---

## Stage 18 — Email marketing (Resend)

**Goal:** the email marketing feature from the demo (this part was cut from the video, so it's designed here).
**Mode:** **Plan → review → Approve.**

```text
Build /email-marketing (owner only).

Layout: 2 columns on desktop.
Top right of the page: a badge "{credits} credits" from subscriptions.credits (live-updated after sends).

Left — "Leads" table of all customers across the owner's domains: checkbox, Email, Domain, Answers ("View" button opens a Dialog listing each filter question and the customer's answer or "—"), Unsubscribed badge if customers.unsubscribed. Search by email and filter by domain. Selecting rows enables "Add to campaign" (choose a campaign in a popover). Leads already in that campaign are skipped (upsert on (campaign_id, customer_id) with ignoreDuplicates, since that pair is unique), then toast "{n} leads added".

Right — Campaigns:
- "Create campaign" button → Dialog with Campaign name → create.
- Campaign cards: name, created date, "{n} customers added", selected card has an orange border. Each card has actions: "Edit email" and "Send".
- "Edit email" → Dialog/Sheet with Subject and Body (simple rich-text editor or textarea with a live HTML preview). Support placeholders {{email}} and {{domain}}. Save to campaigns.subject / template_html.
- "Send" → confirm dialog "Send to {n} customers? This uses {n} credits." → edge function "send-campaign" (auth): verifies the caller owns the campaign AND that every recipient customer belongs to one of the caller's domains (campaign_customers RLS only checks the campaign); skips unsubscribed customers and ones already sent (campaign_customers.sent_at); checks credits >= recipients else error "Not enough credits — upgrade your plan"; sends one email per recipient through Resend from FROM_EMAIL (batch API if available, otherwise loop with small delay) with the template placeholders filled and an unsubscribe footer link {APP_URL}/unsubscribe?c={customer_id}&t={token} (token = HMAC-SHA256 of customer_id keyed with a new UNSUBSCRIBE_SECRET secret, any long random string — ask me to add it; the /unsubscribe edge function recomputes it and compares in constant time); marks sent_at; decrements credits by the number actually sent (atomically, in SQL); returns { sent, skipped }. Toast "Campaign sent to {sent} customers".
- Public page /unsubscribe: verifies the token via an edge function, sets customers.unsubscribed = true, shows "You've been unsubscribed."
Empty states: "No leads yet — they appear when visitors share their email with your chatbot." / "No campaigns yet".
```

**Done when:**
- Create a campaign, add 2 leads, write the email, send: you receive it, and credits go down by 2.
- Sending again skips them.
- Unsubscribe works and the person is excluded next time.

---

## Stage 19 — Dashboard metrics

**Goal:** the `/dashboard` overview.
**Mode:** Build.

```text
Build /dashboard for the owner (all numbers scoped to the owner's domains, loaded via one RPC/edge function "dashboard-stats" for speed).

Row of 4 metric cards (icon, label, big value; wrap to 2×2 on tablet, 1 column on mobile; semantic colors so dark mode works):
- Potential Clients — count of customers.
- Pipeline Value — sum of prices of active products (formatted $).
- Appointments — count of bookings (upcoming in small text).
- Total Sales — sum of payments with status 'paid' (formatted $).

Below, two columns:
- Left "Plan Usage" — "A detailed overview of your metrics, usage, customers and more": progress bars with "used / limit" labels for Email credits (used this period = plan credits − remaining), Domains (count / limit or "Unlimited"), Contacts (customers / limit). Bars turn orange at 80% and red at 100%. Plan name badge + "Upgrade" link if not ultimate.
- Right "Recent Transactions" with a receipt icon and a "See more" link: last 10 paid payments (customer email, domain, amount, date). Empty state "No transactions yet".
Skeleton loaders while loading.
```

**Done when:** the numbers match what's in the database after Stages 10–18.

---

## Stage 20 — SEO blog (built-in CMS, or WordPress)

**Goal:** replace the Cloudways headless WordPress with something simpler that still ranks.
**Mode:** Build. Pick **one** option.

**Option A — built-in CMS (recommended):**

```text
Build an SEO blog with the blog_posts table.
- Make me an admin: tell me the SQL to insert my user id into user_roles with role 'admin' (I'll run it in the SQL editor).
- /admin/blog (admin only, inside the owner layout, extra sidebar item only visible to admins): list posts (title, status, date) with "New post". Editor page: title, slug (auto from title, editable, unique), excerpt, cover image upload (blog-images bucket), rich-text content (headings, bold, links, lists, images) stored as sanitized HTML, Published switch, published_at. Preview button.
- Public /blog: grid of published posts (cover, date like "May 28, 2024", title, excerpt) sorted newest first. Public /blog/:slug: cover, title, date, rendered content (sanitize the HTML on the server with a sanitizer that doesn't need a browser DOM, e.g. sanitize-html — plain DOMPurify breaks server rendering — so the post body is in the server-rendered HTML; styled with Tailwind typography prose classes, images rounded and full width), back link.
- The landing page "News Room" section shows the 3 latest published posts (cards link to /blog/:slug).
- SEO: unique <title>, meta description, canonical URL, og:title/og:description/og:image per post; JSON-LD BlogPosting on post pages; serve sitemap.xml from a server route generated at request time from published blog_posts (plus / and /blog), so new posts appear without a code change; robots.txt allows everything except /dashboard, /settings, /conversation, /admin, /portal, /chatbot.
```

**Option B — keep WordPress:** connect the **WordPress (self-hosted)** connector (site URL, username, application password), then:

```text
Use the WordPress connector as a headless CMS: fetch published posts (with featured media URL and author name) from wp/v2/posts server-side, render /blog and /blog/:slug like the built-in option (look the post up by its WordPress slug; sanitize the HTML on the server with sanitize-html, not DOMPurify, so the post body is in the server-rendered HTML; prose styling, per-post meta/OG tags; sitemap.xml generated at request time from the published posts), and show the 3 latest on the landing page News Room. Cache responses for 5 minutes.
```

**Done when:**
- A published post shows on the home page and on `/blog/:slug`.
- "View page source" on the published site shows the post's title and meta tags (SSR is working).

---

## Stage 21 — Hardening, SEO, publish, custom domain

**Mode:** Build, plus manual steps.

```text
Do a production-readiness pass, then summarize what you changed:
1. Run the security scan and fix all findings. Double-check: RLS on every table; the widget/portal edge functions can only touch rows for the given domain and token/customer; no secrets in client code; storage write policies limited to the owner's own folder ({userId}/… in domain-icons, bot-icons, product-images and blog-images; in chat-uploads, {domainId}/… of a domain the user owns, which the inbox composer needs, so keep that policy); rate-limit chatbot-message per room/IP (e.g. 20 messages/minute) to protect AI credits.
2. Every page: loading skeletons, empty states, error states with retry; consistent toasts.
3. Dark mode audit: find any hard-coded colors and replace them with semantic tokens (sidebar, cards, chat bubbles, code snippet block).
4. Mobile audit at 375px: sidebar, conversations (list → detail with back button), tables scroll horizontally, portal steps.
5. Run the SEO & AI search review and fix: titles/descriptions per public route, OG images, sitemap.xml, robots.txt (private routes disallowed), canonical URLs.
6. Accessibility: labels on all inputs, focus states, alt text, aria-labels on icon buttons, keyboard-friendly widget (Esc closes).
7. Optional: a scheduled Job on the 1st of each month that resets subscriptions.credits to each plan's monthly allowance for users without a Stripe subscription (standard plan).
```

Manual:
1. **Publish.** Set the `APP_URL` secret to the final URL.
2. **Settings → Domains**: connect your custom domain and update `APP_URL` again.
3. Stripe: register the webhook URL and copy `STRIPE_WEBHOOK_SECRET`. When you go live, swap to live keys and live price IDs.
4. Run the full test script in [`05-testing-and-debugging.md`](05-testing-and-debugging.md).
