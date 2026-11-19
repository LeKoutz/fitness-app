# PRD

## What we are building

A fitness web app that can be installed on a phone (a PWA). A coach keeps a library of exercises, and builds each client's workout plan and meal plan. Each client sees only their own plan, in a simple screen with two tabs: workout and meals.

## Who it's for

- **Coach:** manages clients, builds plans and assigns them. Mostly uses a computer.
- **Client:** follows the plan the coach gave them. Mostly uses a phone.

V1 has one coach. Every client, exercise and plan still records which coach it belongs to, so more coaches can be added later without changing how the data is stored.

## User stories

1. As a coach, I want a way to register one client under me.
2. As a coach, I want to have a place to store exercises, workouts, and meal plans.
3. As a coach, I want an easy way to assign plans to clients.
4. As a client, I want a minimal way to see my workout and meal plan without too much clutter.
5. As a client, I want to see which workout I did last, so I know which one to do next.
6. As a coach, I want to build any kind of plan: a 4-day plan, push-pull or push-pull-legs for workouts, and the same diet every day or a different one on training and rest days for meals.

## V1 features

### Accounts

- V1's coach account is created once by the developer in the Supabase dashboard, with a temporary password the developer gives the coach. There's no sign-up page in V1: the coach is set up by hand, and clients only get an account through an invite.
- The coach and clients sign in on the app's sign-in page with their email and password. The coach never sees the Supabase dashboard.
- Once signed in, the coach and clients can change their password.
- The app sends no emails in V1. Invite and password reset links are made by the app and shared by the coach (for example over WhatsApp or SMS).
- **Forgotten password:** a client asks their coach. On the client's page, the coach clicks "Reset password" and gets a link to send them. The client opens it and sets a new password. If the coach forgets their own password, the developer resets it in the Supabase dashboard.
- A reset link that is old or already used shows: "This link has expired or was already used. Ask your coach for a new one."

### Clients (story 1)

- The coach invites a client by entering their name and email. The app creates the client's account and shows an invite link with a "Copy" button. The coach sends the link to the client however they like. The client opens it, sets their password and signs in.
- While a client is invited, the coach can make a fresh invite link (the old one stops working) or cancel the invite, which removes the client (for example after a typo in the email).
- A client who opens an invite link that is old or already used sees: "This invite link has expired or was already used. Ask your coach to send a new one."
- For each client the coach sees:
  - **Profile:** name, email, and a free-text notes field (goals, injuries, preferences)
  - **Plans:** the client's workout plan and meal plan
  - **Last completed:** the workout the client last marked as done, and when (for example "Last completed: Day 2 · Pull, Tuesday")
  - **Status:** invited (hasn't accepted yet), active, or archived (no longer coached, hidden from the main list but kept)
- An archived client who signs in sees a short message and no plans: "Your coaching has ended. Contact your coach to start again."
- The coach can make an archived client active again. Nothing is deleted when a client is archived, so their plans come back.
- The client list marks an active client "No plan" while their workout plan or meal plan is empty, so the coach knows who still needs one.

### Exercise library (story 2)

- **Exercise:** name, short instructions, and a video or image link.
- The coach builds each client's workouts from these exercises.
- Editing an exercise (its name, instructions, or video or image link) changes it in every plan that uses it. What the coach set for each client (sets, reps or time, rest, note) stays in that client's plan and doesn't change.
- Removing an exercise hides it: it's gone from the library and from the list the coach picks from, but plans that already use it still show it to the client. It stays in the database.
- In the coach's plan editor, an exercise that was removed from the library is marked "removed from library" so the coach can swap it.

### Plans (stories 2, 3 and 6)

- Each client has one workout plan and one meal plan. Each client's plans are their own: changing them never changes another client's plans.
- Plans have no dates. The client follows the same plan until the coach changes it. To switch a client to another plan (for example from a 4-day plan to push-pull-legs), the coach edits the plan. The old plan isn't kept.
- **Workout plan:** an ordered list of workouts. The coach adds, removes, renames and reorders them.
  - A new workout is named "Day 1", "Day 2" and so on. The coach can rename it (for example "Push", "Pull", "Legs").
  - A workout is a list of exercises from the library. Each exercise has sets, reps (or time), rest, and an optional note (for example "slow on the way down").
  - There are no rest days in the list: the client does the next workout whenever they train.
- **Meal plan:** a list of day types. A new meal plan has one day type, "Every day". The coach can rename it and add more (for example "Training day" and "Rest day").
  - Each day type has a list of entries: meals, supplements, and pre/post-workout.
  - Each entry has a name, a list of foods with amounts (for example "oats, 80 g"), and an optional note.
  - Each food can list alternatives (for example "chicken 150 g, or turkey 150 g").
  - No calories or macros in V1.
- The app never fills a plan on its own. A plan stays empty until the coach builds it.

### Client screens (stories 4 and 5)

- Two tabs: **Workout** and **Meals**.
- **Workout tab:** a tab for each workout at the top, in the coach's order (for example "Day 1", "Day 2", "Day 3"), and the selected workout's exercises under them. The workout the client last marked as done has a tick on its tab. The tab after it is shown when the app opens (the first one if nothing is marked yet, and the first one again after the last).
- Each exercise shows its name, sets and reps (or time), rest, and its note if there is one (for example "Bench press · 4 × 8 · rest 90 s · slow on the way down"). Tapping an exercise opens its instructions and its video or image link.
- **Mark as done:** under the exercises, a "Mark as done" button. It moves the tick to that workout's tab and records the time. Marking another workout moves the tick again, which also undoes a wrong tap.
- **Meals tab:** with one day type, it shows its entries. With more than one, a tab for each day type at the top (for example "Training day" and "Rest day"), with the first one shown. Each entry shows its foods and amounts, with each food's alternatives and the entry's note under it.
- Apart from "Mark as done", clients only read their plans in V1.
- When a plan is empty, the client sees a short message instead of an empty screen: "No plan yet. Your coach will add one soon."

### Connection

- The app needs a connection in V1. It can still be installed on the phone's home screen.

### Devices

- Client screens are designed for a phone first.
- Coach screens are designed for a computer, and still work on a phone.

### Language

- The app's own text (buttons, messages, labels) is in English only in V1. The coach can write plans, foods and notes in any language.

## Later versions

- Saved workout plans and meal plans that the coach builds once and assigns to many clients, either copied into each client's plan or linked to the saved plan with changes per client (like components in Penpot or Figma). Stories for this:
  - As a coach, I want to be able to save workout plans.
  - As a coach, I want an easy way to assign my pre-made plans to my clients.
  - As a coach, I want to be able to overwrite a days' plan only for a certain client without affecting the default plan.
- Offline: the app opens without a connection and shows the plans the client last loaded, read only, with a banner showing when they were last updated (for example "Offline, last updated Tuesday 18:40"). A "Mark as done" made offline is saved on the phone and sent when the connection comes back.
- Clients record more of what they did: enter weights and keep personal records.
- Body weight, age and progress photos in the client profile.
- Calories and macros for meals, possibly calculated from a food database.
- More than one coach, with a sign-up page for coaches.
- The app's text in Greek as well as English.
- Invite and password reset links sent by email, with a "Forgot password?" link on the sign-in page. This needs an email-sending service and a domain of our own.

## Not decided yet

- The app's name.
