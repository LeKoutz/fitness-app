# PRD

## What we are building

A fitness web app that can be installed on a phone (a PWA). A coach keeps a library of exercises, and builds each client's workouts and meals week by week. Each client sees only their own plan for the day, in a simple screen with two tabs: workout and meals.

## Who it's for

- **Coach:** manages clients, builds plans and assigns them. Mostly uses a computer.
- **Client:** follows the plan the coach gave them. Mostly uses a phone.

V1 has one coach. Every client, exercise and plan still records which coach it belongs to, so more coaches can be added later without changing how the data is stored.

## User stories

1. As a coach, I want a way to register one client under me.
2. As a coach, I want to have a place to store exercises, workouts, and meal plans.
3. As a coach, I want an easy way to assign plans to clients.
4. As a client, I want a minimal way to see my workout and meal plan for the day without too much clutter.
5. As a client, I want a calendar view with my plans for the week/month.
6. As a coach, I want an easy way to carry-over the plan from the previous week to the next instead of creating it again.

## V1 features

### Accounts

- V1's coach account is created once by the developer in the Supabase dashboard, with a temporary password the developer gives the coach. There's no sign-up page in V1: the coach is set up by hand, and clients only get an account through an invite.
- The coach and clients sign in on the app's sign-in page with their email and password. The coach never sees the Supabase dashboard.
- Once signed in, the coach and clients can change their password.
- The app sends no emails in V1. Invite and password reset links are made by the app and shared by the coach (for example over WhatsApp or SMS).
- **Forgotten password:** a client asks their coach. On the client's page, the coach clicks "Reset password" and gets a link to send them. The client opens it and sets a new password. If the coach forgets their own password, the developer resets it in the Supabase dashboard.
- A reset link that is old or already used shows: "This link has expired or was already used. Ask your coach for a new one."

### Clients (story 1)

- The coach invites a client by entering their email. The app creates the client's account and shows an invite link with a "Copy" button. The coach sends the link to the client however they like. The client opens it, sets their password and signs in.
- While a client is invited, the coach can make a fresh invite link (the old one stops working) or cancel the invite, which removes the client (for example after a typo in the email).
- A client who opens an invite link that is old or already used sees: "This invite link has expired or was already used. Ask your coach to send a new one."
- For each client the coach sees:
  - **Profile:** name, email, and a free-text notes field (goals, injuries, preferences)
  - **Plans:** the client's weeks of workouts and meals, and their calendar as the client sees it
  - **Status:** invited (hasn't accepted yet), active, or archived (no longer coached, hidden from the main list but kept)
- An archived client who signs in sees a short message and no plans: "Your coaching has ended. Contact your coach to start again."
- The coach can make an archived client active again. Nothing is deleted when a client is archived, so all their weeks come back.
- The client list warns the coach when a client has no plan for next week yet, so they can build it in time. The warning shows for as long as next week is empty, until the coach builds it or carries one over.

### Exercise library (story 2)

- **Exercise:** name, short instructions, and a video or image link.
- The coach builds each client's workouts from these exercises.
- Editing an exercise (its name, instructions, or video or image link) changes it in every week that uses it, past weeks included. What the coach set for each client (sets, reps or time, rest, note) stays in that client's week and doesn't change.
- Removing an exercise hides it: it's gone from the library and from the list the coach picks from, but weeks that already use it still show it to the client. It stays in the database.
- If carry-over copies a week that uses a removed exercise, the copy keeps it, and the coach's week editor marks it "removed from library" so the coach can swap it.

### Weekly plans (stories 2, 3 and 6)

- The coach builds each client's plans one week at a time. Each client's week is their own: changing it never changes another client's plans.
- A week runs Monday to Sunday, everywhere in the app: the coach's week editor, the date bar and the month view.
- "Today" is the date on the client's phone. A plan belongs to a date, not a time of day, so the coach and client see it on the same date even in different time zones.
- **Workout:** for each day of the week, a workout or a rest day. A workout is a list of exercises from the library. Each exercise has sets, reps (or time), rest, and an optional note (for example "slow on the way down").
- **Meals:** for each day of the week, a list of entries: meals, supplements, and pre/post-workout.
  - Each entry has a name, a list of foods with amounts (for example "oats, 80 g"), and an optional note.
  - Each food can list alternatives (for example "chicken 150 g, or turkey 150 g").
  - No calories or macros in V1.
- **Carry-over:** a button that fills a client's next week with a copy of the previous week, which the coach can then change. The coach can also build a week from scratch at any point, for example to switch a client from push-pull-legs to a 5-day plan.
- The app never fills a week on its own. A week stays empty until the coach builds it or carries one over.

### Client screens (stories 4 and 5)

- Two tabs: **Workout** and **Meals**.
- **Workout tab:** each exercise shows its name, sets and reps (or time), rest, and its note if there is one (for example "Bench press · 4 × 8 · rest 90 s · slow on the way down"). Tapping an exercise opens its instructions and its video or image link.
- **Meals tab:** each entry shows its foods and amounts, with each food's alternatives and the entry's note under it.
- A date bar at the top, shared by both tabs. Today is shown by default. Tapping a day shows that day's workout and meals. The full month opens from the date bar.
- Clients only view their plans in V1. They don't record anything.
- On a day with no plan (for example when the coach hasn't built that week yet), the client sees a short message instead of an empty screen: "No plan for this day yet. Your coach will add one soon."

### Offline

- The app opens without a connection.
- The client can see the plans they last loaded (at least the current and next week), read only, with a banner showing when they were last updated (for example "Offline, last updated Tuesday 18:40").
- The coach screens need a connection.

### Devices

- Client screens are designed for a phone first.
- Coach screens are designed for a computer, and still work on a phone.

### Language

- The app's own text (buttons, messages, labels) is in English only in V1. The coach can write plans, foods and notes in any language.

## Later versions

- Saved workout plans and meal plans that the coach builds once and assigns to many clients, either copied into each client's week or linked to the saved plan with changes per client (like components in Penpot or Figma). Stories for this:
  - As a coach, I want to be able to save workout plans.
  - As a coach, I want an easy way to assign my pre-made plans to my clients.
  - As a coach, I want to be able to overwrite a days' plan only for a certain client without affecting the default plan.
- Clients record what they did: tick workouts as done, enter weights, and keep personal records.
- Body weight, age and progress photos in the client profile.
- Calories and macros for meals, possibly calculated from a food database.
- More than one coach, with a sign-up page for coaches.
- The app's text in Greek as well as English.
- Invite and password reset links sent by email, with a "Forgot password?" link on the sign-in page. This needs an email-sending service and a domain of our own.

## Not decided yet

- The app's name.
