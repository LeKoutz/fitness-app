# ADR

Decisions about how the app is built, and why. Each one lists what we chose, why, what
we didn't choose, and what to watch for.

## 1. Supabase as the backend

Date: 2026-11-11

**Decision:** use Supabase for the database, sign-in, file storage and server functions.
The frontend is plain JavaScript and talks to Supabase through the `supabase-js` library.

**Why:**

- It's open source. If the hosted service stops fitting, the same software can be run on
  our own server.
- It looked simpler than Firebase.
- The free plan should cost nothing until the project really grows. Its limits:
  500 MB database, 50,000 monthly active users, 1 GB file storage, 5 GB egress,
  500,000 Edge Function calls a month, 2 active projects.
- Edge Functions (needed for invites, see decision 2) are part of the free plan.
- It uses Postgres, a SQL database. Our data has many links (a coach has clients, a
  client has plans, a plan's workouts point to exercises in the library), and a SQL
  database is built for that.

**Not chosen:**

- **Firebase:** not open source, and its server functions (Cloud Functions) are only on
  the pay-as-you-go Blaze plan, not on the free Spark plan. Invites would need Blaze
  and a card on file.
- **Appwrite:** open source with a similar free plan, but it also pauses projects after
  a week, and a reply on its forum says app traffic doesn't count as activity, only
  development work. That's worse for a live app.
- **PocketBase:** open source and written in Go, but it has no free hosting. We would
  run and look after our own server, at about $5 a month. It's the fallback if Supabase
  stops fitting.

**Watch for:**

- "Free projects are paused after 1 week of inactivity." Supabase's FAQ says requests
  from the app and dashboard visits count as activity, so normal use by the coach or
  clients keeps the project awake. It only pauses if nobody uses the app for a whole
  week. Supabase emails a warning about a week before pausing, and a paused project
  can be restored from the dashboard.
- Some developers reported in 2026 that read-only requests no longer reliably count as
  activity. This isn't confirmed in Supabase's docs.
- Only a paid plan prevents pausing for sure.

Sources: [pricing](https://supabase.com/pricing),
[pausing FAQ](https://supabase.com/kb/faqs/platform/free-project-pausing),
[Firebase pricing](https://firebase.google.com/pricing),
[Appwrite pricing](https://appwrite.io/pricing),
[Appwrite pausing thread](https://appwrite.io/threads/1481574986136158322).

## 2. Invite and reset links are made by an Edge Function and shared by the coach

Date: 2026-11-11

**Decision:** the app sends no emails in V1. The coach's "Invite" and "Reset password"
buttons call a Supabase Edge Function (a small function Supabase hosts and runs). The
function checks that the caller is the coach, then calls
`supabase.auth.admin.generateLink` with `type: 'invite'` for a new client or
`type: 'recovery'` for a password reset. That method "Generates email links and OTPs to
be sent via a custom email provider" and sends nothing itself, so the function returns
the link and the app shows it with a "Copy" button. The coach sends it to the client
over WhatsApp, SMS or anything else.

The link opens a page in our app, and it's only used when the person presses a button
there (for example "Set my password"). Messaging apps open links to show a preview,
and some email services open every link to check it, which can use up a one-time link
before the person clicks it. The problem and the fix come from developers' reports, not
from Supabase's docs.

**Why an Edge Function:** "Any method under the `supabase.auth.admin` namespace requires
a `secret` key", these methods "should be called on a trusted server", and "Never
expose your `secret` key in the browser." Anyone who has the secret key can read and
change all the data, so it can't be in the frontend code.

**Why no emails:** Supabase's built-in email "will refuse to deliver messages to
addresses that are not part of the project's team", is limited to "2 messages per
hour", and "is not meant for production use". Sending to clients needs a custom SMTP
provider, and the free options either send from a Gmail address (spam risk, and a
personal address on every email) or need a domain of our own, which costs money every
year. Since 3 June 2026, new free projects on the built-in email also can't change
their email templates. Sharing links costs nothing, and the coach already has a way to
reach each client.

**Language:** the function is written in TypeScript, kept close to plain JavaScript.
Edge Functions run on Deno ("Deno compatible runtime with TypeScript first"), and the
docs show functions as "`.ts` files that export a handler". The rest of the app stays
plain JavaScript (decision 8). The function runs locally with `supabase functions serve`.

**Not chosen:**

- **`inviteUserByEmail` with an email provider (Brevo or Resend):** the usual way, but
  needs custom SMTP and, to be reliable, a domain. It's the path for a later version:
  the same links, sent by email.
- **Brevo with a Gmail address as the sender:** free, but more likely to land in spam,
  and every client email would come from a Gmail account.
- **Switching to Firebase,** whose Auth sends emails on the free plan ("Password reset:
  150 emails/day"): creating an account for a client is an admin action, and Firebase's
  server functions need the paid Blaze plan. It would also undo decision 1.
- **The client signs up alone and enters a code from the coach:** no server code, but
  the PRD chose invites.

**Watch for:**

- Links expire after the time set in Email OTP Expiration, which defaults to 1 hour,
  and there's no separate setting for each link type. We set it to the maximum, 1 day
  (86,400 seconds), in the dashboard. A client can still miss the day, which is why the
  PRD lets the coach make a fresh link.
- Links work once. A second use, or a link replaced by a newer one, fails with "Email
  link is invalid or has expired".
- The coach's own account is created by hand, since the dashboard's "Send invitation"
  goes through the built-in email, which can't reach the coach. In the dashboard:
  Authentication > Users > Add user > Create new user, with the coach's email, a
  temporary password and "Auto Confirm User" ticked. Then a row in the coaches table
  marks that user as the coach. The coach signs in and changes the password in the app.
- **Check when creating the Supabase project:** the official docs only describe "Send
  invitation" in the dashboard. "Create new user" with a password and "Auto Confirm
  User" comes from older GitHub reports, so the current dashboard may differ. If it
  isn't there, run `auth.admin.createUser` once from a small script on the developer's
  computer, with the email, the password and `email_confirm: true`. It sends no email.
  The secret key stays on that computer. The same script could make an invite link with
  `generateLink` instead, so the coach joins like a client.

Sources: [generateLink](https://supabase.com/docs/reference/javascript/auth-admin-generatelink),
[admin API](https://supabase.com/docs/reference/javascript/admin-api),
[custom SMTP](https://supabase.com/docs/guides/auth/auth-smtp),
[email template change](https://supabase.com/changelog/46599-changes-to-email-template-customisation-on-free-tier),
[Users](https://supabase.com/docs/guides/auth/users),
[createUser](https://supabase.com/docs/reference/javascript/auth-admin-createuser),
[dashboard Create new user](https://github.com/supabase/auth/issues/1226),
[passwordless sign-in](https://supabase.com/docs/guides/auth/auth-email-passwordless),
[Firebase Auth limits](https://firebase.google.com/docs/auth/limits),
[a second use fails](https://github.com/superserve-ai/superserve/pull/547),
[scanners use up links](https://github.com/George112tech/tru-data/pull/45).

## 3. Every record stores its coach

Date: 2026-11-11

**Decision:** clients, exercises and plans each store the coach they belong to,
even though V1 has only one coach. Who can read and change which rows is enforced by
Row Level Security, see decision 6.

**Why:** adding more coaches later won't need a change to how the data is stored.

**Not chosen:** leaving the coach out until there's a second coach. Less to write now,
but adding it later means changing every table and the data already in them.

## 4. Each client has one workout plan and one meal plan, with no dates

Date: 2026-11-19. Replaces the decision of 2026-11-11 to build plans week by week.

**Decision:** in V1 there are no saved workout or meal plans, and plans have no dates.
Each client has:

- **A workout plan:** an ordered list of workouts. Each workout has a name ("Day 1",
  "Day 2" by default, which the coach can change to "Push" for example) and a list of
  exercises from the library with sets, reps or time, rest and a note.
- **A meal plan:** a list of day types ("Every day" by default, or for example
  "Training day" and "Rest day"). Each day type has its meal entries.
- **Last completed:** one row per client with the workout they last marked as done and
  the time. Marking another workout replaces the row. It points at the workout's id, so
  renaming or reordering workouts keeps the tick on the right one.

The coach edits the plans in place. Old versions of a plan aren't kept.

**Why:**

- Coaches give splits (a 4-day plan, push-pull, push-pull-legs), not calendars. A
  client trains when they can, so a plan tied to dates goes wrong as soon as they skip
  a day. A list of workouts with a tick on the last completed one fits that.
- The same shape serves both: a list of named items, each with its entries. One editor
  pattern for the coach and one tab pattern for the client.
- It's simpler than weeks: no carry-over, no "next week is empty" warning, no date bar
  or month view, and no time zone rules.
- It doesn't block saved plans later. They can be added as "start from a saved plan",
  which copies the saved plan into the client's plan, so the plans stay the same shape.

**Not chosen:**

- **Plans built week by week, with dates** (the decision of 2026-11-11): the coach had
  to build or carry over every client's week, every week, and the client only needed
  the calendar to find today's workout.
- **The app working out today's workout from the date** (for example "every Monday is
  Day 1"): breaks when a client skips or moves a day. The tick on the last completed
  workout lets the client decide.
- **Keeping old plans as history:** nobody needs it in V1. Saved plans in a later
  version would cover it.
- **Saved plans linked to each client, with changes per client** (like components in
  Penpot or Figma): one edit to a saved plan reaches every client who follows it, but it
  needs links, changes per client, and rules for what an edit does to each client.
  Moving from this model back to plans per client would mean undoing the links.
- **Saved plans copied into each client's plan:** a library editor on top of plans per
  client. Left for a later version, since it can be added without changing how plans
  are stored.

**Watch for:**

- The coach types every client's plans by hand. If coaches end up building the same
  plans for many clients, that's the sign to add saved plans.
- If the coach removes the workout that's marked "Last completed", the row goes with
  it and the client sees no tick until they mark another workout.

## 5. Offline with a service worker and a saved copy

Date: 2026-11-11. Moved to a later version on 2026-11-19.

**V1:** the app needs a connection, and there is no service worker. It can still be
installed: Chrome's install criteria are HTTPS and a web app manifest (a name, icons at
192 and 512 px, a `start_url` and a `display` value), and they don't list a service
worker. Moved because "Mark as done" (decision 4) is the first thing a client sends to
the database, and sending it later from offline needs its own logic: a retry when
sending fails (the browser's "online" signal isn't reliable on weak wifi), the time of
the tap rather than of the send, dropping a tap whose workout the coach removed, and
dropping waiting taps on sign out.

Source: [install criteria](https://web.dev/articles/install-criteria).

The rest of this decision is the plan for that later version.

**Decision:**

- A service worker (a script the browser runs in the background for the site) saves the
  app's own files (HTML, CSS, JS), so the app opens without a connection.
- Each time the client loads their plans, the app saves a copy on the phone. When
  offline, it shows that copy, read only, with the time it was last updated.
- A "Mark as done" made offline is saved on the phone and sent when the connection
  comes back.
- Sign-in stays saved through `auth.persistSession`, which "automatically save[s] the
  user session into local storage".

**Why:** clients often open the app in gyms with bad signal.

**Not chosen:** no offline support. Less work, but the app would fail when it's most
needed.

**Watch for:**

- `supabase-js` doesn't seem to save data for offline use on its own. Its client options
  page lists nothing for it, so we write the saved copy ourselves.
- Where on the phone to save the copy (localStorage or IndexedDB) is not decided yet.
- Offline gets harder as clients record more (weights, records), since all of it has
  to be sent back later.

Source: [createClient options](https://supabase.com/docs/reference/javascript/initializing).

## 6. Row Level Security on every table

Date: 2026-11-11

**Decision:** every table has Row Level Security turned on, with policies (rules written
in SQL and attached to a table) for who can read and change its rows. A coach reads and
changes only their own data. A client reads their own plans, and writes only their own
"Last completed" row, which must point at a workout in their own plan. Supabase checks the
policies on every request, whatever the frontend code does.

**Why:** the frontend talks to the database directly, using a public key that anyone
can see in the browser. The policies are what actually stop a client from reading
another client's plans.

**Not chosen:** checking permissions in an Edge Function for every request. That works,
but every read would need server code, which is more to build and slower.

**Watch for:** policies are written as migration files in the repo and tested before
they reach the real project, see decisions 9 and 10.

## 7. Cloudflare Workers hosts the frontend

Date: 2026-11-11

**Decision:** the frontend (HTML, CSS, JS) is served as static assets on Cloudflare
Workers, on the Free plan, deployed from the GitHub repo.

**Why:**

- The app is only static files. Everything dynamic runs on Supabase (decision 1), so
  static hosting is enough.
- "Requests to static assets are free and unlimited" on Workers.
- Every site gets an HTTPS address, which installing the app needs (decision 5).
- Cloudflare recommends Workers over Pages for new projects: "Workers supports most
  Pages use cases and offers a broader feature set" and "Start new projects with
  Workers."

**Not chosen:**

- **Cloudflare Pages:** generous free plan (500 builds a month, 20,000 files per site),
  but Cloudflare now points new projects to Workers.
- **Vercel:** "Our Hobby plan is for personal, non-commercial use", so it would have to
  change if the project ever makes money.
- **GitHub Pages:** not allowed for "your online business" or "commercial software as a
  service (SaaS)", for the same reason.
- **Netlify:** a "300 credit limit" on the free plan, with bandwidth at "20 credits per
  GB", and the pricing page doesn't say what happens at the limit.

**Watch for:**

- The Workers Free plan has a limit of 100,000 requests a day. The pricing page doesn't
  say outright whether static asset requests count toward it. Since they're described
  as free and unlimited, most likely they don't, and the app runs no code on Workers.
- This is a free project for now. Whether the Free plan allows business use wasn't
  checked; check it before the project starts making money.

Sources: [Workers pricing](https://developers.cloudflare.com/workers/platform/pricing/),
[Pages](https://developers.cloudflare.com/pages/),
[Pages limits](https://developers.cloudflare.com/pages/platform/limits/),
[Vercel pricing](https://vercel.com/pricing),
[GitHub Pages limits](https://docs.github.com/en/pages/getting-started-with-github-pages/github-pages-limits),
[Netlify pricing](https://www.netlify.com/pricing/).

## 8. Plain JavaScript with Vite, no framework

Date: 2026-11-11

**Decision:** the frontend is written in plain JavaScript, with no framework. Vite runs
the local dev server and bundles the files for deploying (decision 7). Packages like
`supabase-js` are installed with `npm`.

**Why:**

- The screens are mostly forms and lists, which plain JavaScript handles well.
- Without a framework, the code uses the browser's own features directly, with no
  extra rules on top.
- Vite is the usual way to use `npm` packages in a plain JavaScript project. It reloads
  the page on save while developing, and the code stays plain JavaScript.

**Not chosen:**

- **A framework (React, Vue, Svelte):** helps with large, changing screens, but adds
  its own concepts to learn, and this app's screens don't need it.
- **Plain files with no build tool:** nothing to install, but `supabase-js` would come
  from a CDN link instead of `npm`, and tests would need setting up by hand.

## 9. Vitest for frontend and database tests

Date: 2026-11-11

**Decision:** all tests are written in JavaScript and run with Vitest. Frontend unit
tests test the app's own logic. Database tests sign in as the coach, a client and a
second client through `supabase-js`, and check what each one can and can't read and
change. Database tests run against a local copy of Supabase, never the real project.

**Why:**

- One tool and one language for every test.
- Database tests go "end to end, through the same client your app uses", so they test
  what the app can really reach.
- Vitest comes with Vite (decision 8).

**Not chosen:** pgTAP, "a unit testing framework for Postgres", run with
`supabase test db`. It tests policies from inside the database, in SQL, and would add a
second tool and a second language for tests.

**Watch for:** Supabase's advice for policy tests: test create, read, update and delete,
with different users, and "Always test negative cases: what users should not be able to
do."

Source: [Supabase testing overview](https://supabase.com/docs/guides/local-development/testing/overview).

## 10. Database changes as migration files, tested locally

Date: 2026-11-11

**Decision:** every change to the database (tables, columns, Row Level Security
policies, functions) is written as a migration file in `supabase/migrations`, created
with `supabase migration new <name>`, and committed with the code. It's applied to a
local copy of Supabase with `supabase db reset`, tested there (decision 9), and only
then applied to the real project with `supabase db push --remote`. The real project's
database is never changed by hand. The Supabase CLI is installed in the project with
`npm install supabase --save-dev`. There is one project on Supabase, the real one:
development and tests use the local copy, so the free plan's second project stays
spare.

**Why:**

- Every policy is reviewed in a pull request and kept in git history. For Row Level
  Security (decision 6), that's where mistakes should be caught.
- The local copy gets the same tables and policies as the real project, so the
  database tests run against what will really be deployed.
- Tables can still be made with clicks: "use the Dashboard to create tables and
  columns, and then use the CLI to diff your changes and create migrations" with
  `supabase db diff -f <name>`, on the local copy's dashboard.

**Not chosen:**

- **The real project's dashboard only:** nothing to install, but policies exist only in
  the real database, with no review, no history, and nothing to test against.
- **The dashboard, then `supabase db pull --remote` into files:** files end up in git,
  but only after the change is already live.
- **A second Supabase project for development instead of a local copy:** no Docker
  needed, but it needs a connection, can pause like any free project, and uses the
  second free project.

**Watch for:**

- The local copy "runs in Docker containers, so you need a container runtime installed
  first." The CLI needs "Node.js 20 or later" when run through `npm` or `npx`.
- Changing the real database by hand "bypasses the migration history and will cause
  `db push` to fail with sync errors."

Sources: [database migrations](https://supabase.com/docs/guides/deployment/database-migrations),
[CLI getting started](https://supabase.com/docs/guides/local-development/cli/getting-started).
