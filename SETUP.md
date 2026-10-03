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

## 4. Deploy the security rules
1. Paste the contents of `firestore.rules` (included alongside this file) into **Build →
   Firestore Database → Rules** → **Publish**. No editing needed — unlike some earlier drafts
   of this file, it no longer hardcodes any emails; who's admin and who's viewer lives in
   Firestore itself, set up in step 5.
2. This file is still what actually enforces access — the app's own screens are just convenience
   on top of it.

## 5. Sign in — the first person becomes the founding admin
1. Open the app and sign in (Google or email/password). If this is a brand new project with no
   one set up yet, **the very first person to sign in automatically becomes the founding admin** —
   nothing to configure first. That only ever happens once; after this first sign-in, every
   further sign-in is checked against the real list.
2. As an admin, go to **Settings** to add further Admin or Viewer emails — one per line, under
   each list, **Save** each. These are stored in Firestore itself, shared across every device
   immediately — there's no separate copy to keep in sync per browser.
3. An email that isn't on either list is always blocked at sign-in, with no exception (other
   than the one-time founding-admin bootstrap above).
4. Need to start over? An admin can delete the `roles/config` document directly in the Firestore
   console — the next person to sign in becomes the new founding admin.

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
  the same as every other Firebase project. Step 4's rules, together with the Admin/Viewer list
  in Firestore (step 5), are what actually keep the data private and read-only where it should
  be, not hiding the config.
- If someone's access ever changes — added, removed, or moved between Admin and Viewer — an
  admin updates it once in Settings. Because the list lives in Firestore, every device sees the
  change immediately; there's nothing to keep in sync per browser anymore.
- An email that isn't on either list is always blocked at sign-in — no fallback, no exceptions,
  other than the one-time founding-admin bootstrap described in step 5.
- PDF import (loading a CMS "Student Result Summary" export) is built in — the "Import PDF"
  button in the top bar, admin-only. It reads marks, grade, and credit hour straight from the
  PDF; derives which semester each result belongs to from the student's batch and the exam
  period shown (shown editable in case that guess is ever wrong); and, per row, lets you choose
  Add / Update existing record / Treat as resit / Skip, with a sensible default already picked
  based on what's on file.
