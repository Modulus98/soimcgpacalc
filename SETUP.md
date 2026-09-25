# Setup guide — SoIM CGPA Tracking System

The app works immediately with no setup (local mode, browser-only, seeded with the 736 existing
result records). These steps cover the one-time Firebase project setup and turn on real shared
storage: Firestore, sign-in restricted to specific people, and two access levels — Admin (full
access) and Viewer (read-only).

Your project is already created and its config is baked directly into `index.html`, so every
device and browser auto-connects with no "paste config" step. Steps 1–3 and 6 below are already
done for this project — they're here for reference, or in case you ever need to recreate it or
point the app at a different one (Settings → Firebase project config still lets you override the
baked-in config per browser). Steps 4–5 are the ones that still need your input and stay relevant
any time someone's access changes.

## 1. Create a Firebase project
1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project**.
2. Give it a name (e.g. `soim-cgpa-tracking`). Google Analytics is not needed — you can leave it off.

## 2. Turn on Firestore
1. In the left sidebar: **Build → Firestore Database → Create database**.
2. Choose a region close to you (e.g. `asia-southeast1` for Malaysia).
3. Start in **production mode** (the rules in step 4 replace the default-deny anyway).

## 3. Enable sign-in methods
1. **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Google**. Also enable **Email/Password** if you'd like
   that as a backup option — both work in the app, and neither is required if you don't want it.
3. Google sign-in works with existing Google accounts directly — you don't pre-create users
   for it the way Email/Password needs (that one still needs Users → Add user for each account).

## 4. Deploy the security rules — with your real emails, admins and viewers
This is the step that actually restricts access — nothing else does.
1. Open `firestore.rules` (included alongside this file). Find `isAdmin()` and `isViewer()` near
   the top and replace the placeholder emails with your real ones — HOP/HOS and yourself under
   `isAdmin()`, everyone else under `isViewer()` — exactly as each person will sign in with.
2. **Build → Firestore Database → Rules** tab → paste the edited contents in → **Publish**.
3. Whenever anyone's access changes — added, removed, or moved between the two roles — this file
   is the one that has to be edited and republished; the rest is just app-side convenience.

## 5. Set the same emails in the app
1. Open the app → **Settings**.
2. Under "Admin emails," enter the same admin addresses from step 4, one per line. Under "Viewer
   emails," enter the same viewer addresses. **Save** each.
3. These lists only run in the browser — they're what give someone a clear "you're not
   authorized" message, or the correct read-only view, instead of a wall of confusing errors.
   Step 4 is what actually enforces it. This is also per-browser storage: if both lists are ever
   empty on a given browser (a fresh device that's never had them entered), anyone who signs in
   there is shown as admin by default rather than locked out — harmless, since step 4's rules
   are what actually decide what they can write. Still worth entering the real lists on any
   device you specifically want to show the read-only Viewer experience.

## 6. Register a web app and get your config
1. Project Overview (gear icon) → **Project settings → General**.
2. Under **Your apps**, click the **</>** (web) icon → give it any nickname → **Register app**.
   You do not need Firebase Hosting at this step, just the config object.
3. Copy the `firebaseConfig` object shown (starts with `{ apiKey: ... }`) — this is the object
   already baked into `index.html`, so you only need this again if setting up a new project.

## 7. Sign in
1. Open `index.html` in a browser (or your deployed URL). Since the config is baked in, it
   connects automatically — no Settings step needed here anymore.
2. You'll land straight on the sign-in screen — click **Sign in with Google** and use one of the
   accounts from step 5 (or sign in with email/password if you set that up instead).
3. First time on a fresh project? Back in Settings, click **Import existing records to
   Firestore** once. This writes all 736 existing result records into your project. Only do this
   once per project — running it again will create duplicates.

## 8. Everyday use
Every device and browser auto-connects to this same shared project — nothing to reconnect. Sign
in as an admin to add, edit, or remove results, mark a student Active/Withdrawn/Graduated, upload
a subject-code/credit-hour mapping or a bulk results spreadsheet, or import a CMS PDF result
summary directly (the "Import PDF" button in the top bar). Sign in as a viewer to see all the
same data — every intake, student, and export — with none of those controls available.

## 9. Deploy it somewhere real (optional)
The HTML file is fully self-contained — no build step. Either:
- **Firebase Hosting:** `firebase init hosting` (point the public directory at the folder
  containing this file, named `index.html`), then `firebase deploy`.
- **Vercel:** drag the folder into a new Vercel project, or `vercel deploy` from the CLI. The
  file must be named `index.html` at the root — Vercel serves that for the site's root URL,
  and a different filename there is exactly what caused an earlier 404.

Once deployed, anyone visiting the URL still auto-connects to the same project and still hits the
sign-in screen — the app's behavior doesn't change based on where it's hosted.

## Notes
- The Firebase config object is not a secret — it's meant to be visible in a web app's source,
  the same as every other Firebase project. Steps 4 and 5 (the rules, and the matching Admin/
  Viewer lists) are what actually keeps the data private and read-only where it should be, not
  hiding the config.
- If someone's access ever changes — added, removed, or moved between Admin and Viewer — update
  all three places: `firestore.rules` (redeploy it), and both Settings lists on every browser you
  administer from. Missing the rules file means they can sign in but every read (viewer) or write
  (admin) still fails; missing a browser's Settings lists just means that one browser shows the
  wrong UI until it's updated (see step 5's fallback behavior).
- PDF import (loading a CMS "Student Result Summary" export) is built in — the "Import PDF"
  button in the top bar, admin-only. It reads marks, grade, and credit hour straight from the
  PDF; derives which semester each result belongs to from the student's batch and the exam
  period shown (shown editable in case that guess is ever wrong); and, per row, lets you choose
  Add / Update existing record / Treat as resit / Skip, with a sensible default already picked
  based on what's on file.
