# 05 — End-to-end test script, debugging prompts, and pitfalls from the video

## A. End-to-end test script (run after Stage 21, and again before going live)

Use two browsers, or a normal window and an incognito window. **Owner** = signed into the app. **Visitor** = a plain HTML page on another origin with your embed snippet.

| # | Do this | Expect |
|---|---|---|
| 1 | Visit `/`, toggle dark mode, click Pricing | Clean in both themes; 3 plans from `plans.ts` |
| 2 | Sign up as "I own a business" | Step bar advances; 6-digit code email; lands on `/dashboard` |
| 3 | Add domain `test-shop.com` with an icon | Appears in the sidebar; a second domain on Standard is blocked with the limit toast |
| 4 | `/settings/test-shop.com`: copy the snippet, add 2 FAQs and 3 filter questions (e.g. "What's your name?", "What's your budget?", "When do you want to start?") | Saved; the snippet uses your real URL with no trailing slash |
| 5 | Visitor page: open the bubble | Welcome message; Help Desk tab lists the FAQs |
| 6 | Visitor: "hi, what do you sell?" | Friendly answer using FAQs/products; asks for the email; **no** qualifying questions yet |
| 7 | Visitor: gives an email | A customer is created (Email Marketing → Leads); bot starts asking questions naturally |
| 8 | Answer 2 of the 3 questions | Answers saved (Leads → View) |
| 9 | Visitor: "can I book a call?" | **One** clean appointment link built from `APP_URL` |
| 10 | Open the link | Step 1 shows the 3rd (unanswered) question plus the pre-filled answers; the booking calendar works; booked slot disabled afterwards |
| 11 | Owner `/appointment` | Booking listed; under "today" if booked today |
| 12 | Owner `/integration` → Connect Stripe → test onboarding | Shows **Connected** |
| 13 | Add 2 products (Stage 15), then the visitor asks "I want to buy" → link → pay with `4242 4242 4242 4242` | Thank-you page; payment `paid`; connected account receives the transfer |
| 14 | Visitor: "you're useless, get me your manager" | Handoff message; widget shows "Real time"; owner gets **one** email |
| 15 | Owner `/conversation`: open the room | Urgent icon clears after opening; real-time switch is on |
| 16 | Owner and visitor exchange messages and images | Instant on both sides; **no duplicates**; AI silent |
| 17 | Owner turns real-time off; visitor writes | AI answers again |
| 18 | Email Marketing: campaign → add 2 leads → edit email → send | Emails arrive; credits −2; resend skips them; unsubscribe link works |
| 19 | Settings → Upgrade → Pro (test card) | Plan Pro, credits 50, domain limit 5, bot customisation unlocked |
| 20 | `/dashboard` | Clients, pipeline, appointments, sales, usage bars and transactions all correct |
| 21 | Change the bot colours and icon on Pro, then reload the visitor page | New look in the widget |
| 22 | Publish a blog post | Shows on home and `/blog/:slug`; page source has the title and meta |
| 23 | Sign in as a **second** owner | Sees none of the first owner's data (RLS check) |

## B. Debugging prompts (use Chat mode first, so nothing changes until you agree)

**General bug report:**
```text
Bug: <what I did> → <what I expected> → <what happened>. Page: <route>. Console/edge function log error: <paste>.
First explain the root cause in 2-3 sentences without changing code. Then propose the smallest fix. Only change files related to this bug; don't touch auth, RLS or styling elsewhere.
```

**Edge function failing:** open Cloud → Edge functions → View logs, copy the error, then:
```text
The edge function "<name>" fails with: <paste log>. Input I sent: <json>. Find the cause, fix it, and add input validation + a clear error message the UI can show in a toast.
```

**RLS / "no rows" / permission denied:**
```text
As owner X I get <empty list / permission denied> on <table/page>. List the RLS policies for that table, explain which one blocks this, and fix it without opening access to other owners or anonymous users.
```

**AI behaving badly:**
```text
The chatbot did <bad behaviour> in this conversation: <paste>. Adjust only the system prompt / JSON handling in chatbot-message so that <desired behaviour>. Keep replies under 3 sentences and keep links server-generated.
```

**Undo:** if a prompt went wrong, use **version history → restore** the bookmarked version instead of prompting the AI to "undo".

## C. Pitfalls the video ran into, and how this rebuild avoids them

| Video bug or gotcha | Cause | In this rebuild |
|---|---|---|
| Toast errors invisible during sign-up (1:11) | No `<Toaster/>` mounted | Stage 3 explicitly mounts it |
| "Password found in a data breach" | Clerk breach check | Cloud has the same HIBP check; the Stage 3 prompt maps it to a readable toast |
| Image URL blocked (`next.config` remote patterns) | Next.js image domains | N/A (no Next Image); Cloud Storage URLs |
| Plan named "Ultimate" in the DB but "Unlimited" in pricing cards (8:38) | Two sources of truth | One `plans.ts` and an enum id; never compare display names |
| Second domain blocked on Standard (expected) | Plan limit | DB trigger (server-side, not just UI) |
| Iframe broke because of a trailing slash in the snippet URL (6:32) | `…/chatbot/` | Snippet uses `embed.js` with `window.location.origin` and no trailing slash |
| Duplicate messages in real-time chat (9:03) | Optimistic append plus realtime echo | Dedupe by message `id` on both sides (Stage 11) |
| Pusher created a new client on every hot reload | Dev server and the Pusher singleton | N/A; Cloud Realtime |
| Hard-coded Pusher cluster | Constant vs env | N/A |
| File input reset before the upload finished (9:01) | `reset()` called too early | Prompts say "upload FIRST, then reset" |
| Portal link printed twice or wrapped in markdown (7:00) | The model wrote the URL in markdown | **The server appends links**; the model never writes URLs |
| AI saved the wrong answer to a question (7:03) | `(complete)` keyword heuristic | Structured JSON with `answered_question_id` |
| Questions step didn't save edits (7:05) | Nested forms | One save call for answers plus booking/payment |
| "Today's" appointments never matched (8:50) | Compared timestamps, not dates | Stage 13 compares dates in the user's timezone |
| Empty array treated as "has data" (5:14) | `[]` is truthy | Knowledge rule: check `length` |
| Stripe Connect "update your branding/icon" error (7:55) | Connect branding not set | Stage 0 sets branding first |
| Stripe payments failed for some countries (8:51) | Fake account country/data | Real Express onboarding with Stripe test-data shortcuts |
| Stripe not working in the Lovable preview | Lovable limitation (customer portal and some flows) | Test payments on the **published** URL |
| Plan-usage bar showed the wrong number (8:53) | Wrong variable (remaining vs used) | Stage 19 defines "used = monthly allowance − remaining" |
| Gmail app-password setup | Nodemailer via Gmail | Resend with a verified domain |
| Sign-in page 404 / "default export is not a React component" | Empty page file | N/A; Lovable creates the routes |
