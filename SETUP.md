# Setup guide — SoIM CGPA Tracking System

The app works immediately with no setup (local mode, browser-only, seeded
with the 736 existing result records). These steps are for turning on real
shared storage: Firebase Firestore + sign-in restricted to specific people.

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

## 4. Deploy the security rules — with your real two emails
This is the step that actually restricts access — nothing else does.
1. Open `firestore.rules` (included alongside this file). Find `isAllowed()` near the top and
   replace the two placeholder emails (`you@gmail.com`, `colleague@gmail.com`) with your actual
   two Gmail addresses, exactly as they'll sign in with.
2. **Build → Firestore Database → Rules** tab → paste the edited contents in → **Publish**.

## 5. Set the same two emails in the app
1. Open the app → **Settings → "Google sign-in — allowed emails"**.
2. Enter the *same* two addresses from step 4, one per line → **Save**.
3. This list only runs in the browser — it's what gives someone a clear "you're not authorized"
   message instead of a wall of confusing errors. Step 4 is what actually stops them; keep both
   lists in sync whenever the two people change.

## 6. Register a web app and get your config
1. Project Overview (gear icon) → **Project settings → General**.
2. Under **Your apps**, click the **</>** (web) icon → give it any nickname → **Register app**.
   You do not need Firebase Hosting at this step, just the config object.
3. Copy the `firebaseConfig` object shown (starts with `{ apiKey: ... }`).

## 7. Connect the app
1. Open `index.html` in a browser (or your deployed URL, once you've done step 9).
2. Click **Settings**, paste the config object from step 6 into the text box, click **Connect**.
3. You'll be dropped on the sign-in screen — click **Sign in with Google** and use one of the
   two accounts from step 5 (or sign in with email/password if you set that up instead).
4. Back in Settings, click **Import existing records to Firestore** once. This writes all 736
   existing result records into your new project. Only do this once per project — running it
   again will create duplicates.

## 8. Everyday use
From here on, every browser that has this same Firebase config saved (via Settings → Connect)
and signs in as one of the allowed accounts sees and edits the same live data. Add, edit, or
remove a result from a student's page; mark a student Active / Withdrawn / Graduated from the
same page; upload a subject-code/credit-hour mapping or a bulk results spreadsheet from Settings.

## 9. Deploy it somewhere real (optional)
The HTML file is fully self-contained — no build step. Either:
- **Firebase Hosting:** `firebase init hosting` (point the public directory at the folder
  containing this file, named `index.html`), then `firebase deploy`.
- **Vercel:** drag the folder into a new Vercel project, or `vercel deploy` from the CLI. The
  file must be named `index.html` at the root — Vercel serves that for the site's root URL,
  and a different filename there is exactly what caused the earlier 404.

Once deployed, anyone visiting the URL still hits the sign-in screen and still needs to be one
of the two allowed accounts — the app doesn't change behavior based on where it's hosted.

## Notes
- The Firebase config object is not a secret — it's meant to be visible in a web app's source,
  the same as every other Firebase project. Steps 4 and 5 (the rules and the matching allowlist)
  are what actually keeps the data private, not hiding the config.
- If a third person ever needs access, add their email to *both* `firestore.rules` (redeploy it)
  and the app's Settings allowlist — missing either one means either they can't get in, or they
  can sign in but every read/write fails.
- PDF import (bulk-loading a CMS result slip) is not wired up yet — see the main chat for that.
