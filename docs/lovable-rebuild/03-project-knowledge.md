# 03 — Project Knowledge (paste into Lovable)

Copy everything **inside the code block** into **Project settings → Knowledge** right after Stage 1 creates the project. It's under Lovable's 10,000-character limit. Lovable sends it with every message, so the agent always knows the product, schema, plans and rules.

Change the name, prices and colours to your own before pasting. If you change them later, update this text too, so knowledge and code don't drift apart.

```markdown
# AIChat — project knowledge

## Product
AIChat is a multi-tenant SaaS: an AI "sales rep" chatbot that businesses embed on their own website with one script tag. The bot greets visitors, captures their email (turning them into a lead), asks the business owner's qualifying "filter questions" naturally, and sends the visitor a link to book an appointment or pay for products. Owners manage everything in a dashboard: domains (one chatbot per domain), bot training (FAQ + filter questions), live chat takeover, appointments, products + Stripe payouts, email campaigns, metrics.

## Users
- Owner: signed-in AIChat user (a business). Owns domains, sees only their own data.
- Visitor/customer: anonymous person in the widget or customer portal. Never signs in. Becomes a "customer" row once they give an email.
- Admin: platform staff (role in user_roles table) who can edit blog posts.

## Stack rules
- Lovable Cloud for database, auth, storage, realtime, edge functions, secrets.
- Tailwind + shadcn/ui + lucide-react. Font: Plus Jakarta Sans.
- Brand: orange accent (#F97316 family), warm cream background (#FDF8F3) in light mode, near-black in dark. Light/dark/system theme. ALWAYS use semantic tokens (bg-background, text-foreground, bg-muted, border, primary...), never hard-coded colors, so dark mode works.
- Forms: react-hook-form + zod; reuse one <FormField> style everywhere. Toast on every success/error.
- Every list/page has loading (skeleton), empty and error states. Check array.length, never truthiness of an array.
- Never call AI, Stripe or Resend from the browser. Only from edge functions using secrets.
- Secrets: STRIPE_SECRET_KEY, RESEND_API_KEY, APP_URL (public base URL of the app), FROM_EMAIL. LOVABLE_API_KEY is automatic.
- Build every public link (portal links, embed snippet) from APP_URL / window.location.origin. Never hard-code localhost or lovable preview URLs.
- Uploads: max 2 MB, png/jpg/jpeg/webp.
- Mobile-first, responsive; sidebar collapses.

## Plans (single source of truth: src/constants/plans.ts + enum plan = 'standard' | 'pro' | 'ultimate')
| plan     | price  | domains   | contacts | email credits / month | bot customization |
| standard | Free   | 1         | 10       | 10                    | no                |
| pro      | $15/mo | 5         | 50       | 50                    | yes               |
| ultimate | $35/mo | unlimited | 500      | 500                   | yes               |
Display names: "Standard", "Pro", "Ultimate". Enforce limits server-side (DB trigger / edge function) AND show them in the UI.

## Data model (public schema, uuid PKs, created_at default now())
- profiles(id = auth.users.id, full_name, account_type 'owner'|'student', stripe_account_id, stripe_charges_enabled bool)
- user_roles(user_id, role 'admin'|'user')  -- roles never stored on profiles
- subscriptions(user_id PK, plan, credits int, stripe_customer_id, stripe_subscription_id, current_period_end)
- domains(id, user_id, name unique, icon_url)
- chatbots(id, domain_id unique, welcome_message, icon_url, background_color, text_color, show_help_desk bool)
- help_desk(id, domain_id, question, answer)
- filter_questions(id, domain_id, question, sort_order)
- customers(id, domain_id, email, unsubscribed bool) unique(domain_id, email)
- customer_responses(id, customer_id, filter_question_id, question, answer nullable)
- chat_rooms(id, domain_id, customer_id nullable, visitor_token uuid, live bool, mailed bool, starred bool, last_message_at)
- chat_messages(id, room_id, role 'user'|'assistant'|'owner', content, image_url, seen bool)
- bookings(id, domain_id, customer_id, email, date, slot text) unique(domain_id, date, slot)
- products(id, domain_id, name, image_url, price_cents int, active bool)
- payments(id, domain_id, customer_id, amount_cents, currency, stripe_session_id unique, status, customer_email)
- campaigns(id, user_id, name, subject, template_html); campaign_customers(campaign_id, customer_id, sent_at)
- blog_posts(id, slug unique, title, excerpt, content_html, cover_url, published bool, published_at)

RLS: owners can only select/insert/update/delete rows that belong to a domain they own (domains.user_id = auth.uid()). Visitors NEVER read or write tables directly: the widget and the customer portal call public edge functions that validate domain_id + visitor_token (widget) or domain_id + customer_id (portal) and use the service role internally.

## Routes
Public: / · /blog · /blog/:slug · /auth/sign-up · /auth/sign-in · /chatbot?id=DOMAIN_ID (widget page, loaded in an iframe on customer sites) · /portal/:domainId/appointment/:customerId · /portal/:domainId/payment/:customerId · /unsubscribe
Owner (auth required, shared layout: collapsible sidebar + top infobar with page title/description): /dashboard · /conversation · /integration · /settings · /settings/:domainName · /appointment · /email-marketing
Admin: /admin/blog

## Chatbot behaviour (edge function "chatbot-message")
1. No customer on the room yet → greet warmly as the business's sales rep and steer toward getting the visitor's email. Answer FAQ questions from help_desk.
2. Message contains an email → upsert customer (domain_id, email), link room, create customer_responses rows for every filter question not yet present.
3. Customer known → ask unanswered filter questions one at a time, conversationally (not like a script). Save answers.
4. Model returns JSON {reply, answered_question_id, answer, intent: none|book_appointment|buy|handoff}. The SERVER appends portal links for book/buy (model never writes URLs).
5. handoff (rude, frustrated, asks for a human, or out of scope) → room.live = true, email the owner once (mailed=true), bot stops replying until the owner turns live off.
6. While live, visitor and owner chat in realtime; messages deduped by id.

## Conventions
- One hook per feature in src/hooks, small components, constants in src/constants.
- Don't rename or drop existing tables/columns; add new ones.
- Ask before changing auth, RLS, payments or deleting data.
- Keep the infobar title in sync with the current route.
```
