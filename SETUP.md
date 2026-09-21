# Setup guide — SoIM CGPA Tracking System

The app works immediately with no setup (local mode, browser-only, seeded
with the 736 existing result records). These steps are for turning on real
shared storage: Firebase Firestore + admin sign-in.

## 1. Create a Firebase project
1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Add project**.
2. Give it a name (e.g. `soim-cgpa-tracking`). Google Analytics is not needed — you can leave it off.

## 2. Turn on Firestore
1. In the left sidebar: **Build → Firestore Database → Create database**.
2. Choose a region close to you (e.g. `asia-southeast1` for Malaysia).
3. Start in **production mode** (the rules in step 4 replace the default-deny anyway).

## 3. Turn on Authentication and create admin accounts
1. **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Email/Password**. Leave every other provider off.
3. Under the **Users** tab, click **Add user** for yourself and anyone else who should have
   admin access (e.g. Dr. Lim). Set a password for each — there is no public sign-up screen,
   so this console is the only place accounts get created.

## 4. Deploy the security rules
This is the step that actually protects the data — the app's sign-in screen alone does not.
1. **Build → Firestore Database → Rules** tab.
2. Replace the contents with everything in `firestore.rules` (included alongside this file).
3. Click **Publish**.

## 5. Register a web app and get your config
1. Project Overview (gear icon) → **Project settings → General**.
2. Under **Your apps**, click the **</>** (web) icon → give it any nickname → **Register app**.
   You do not need Firebase Hosting at this step, just the config object.
3. Copy the `firebaseConfig` object shown (starts with `{ apiKey: ... }`).

## 6. Connect the app
1. Open `soim-cgpa-tracking-system.html` in a browser (or your deployed URL, once you've done step 8).
2. Click **Settings**, paste the config object from step 5 into the text box, click **Connect**.
3. You'll be dropped on the sign-in screen — sign in with one of the accounts from step 3.
4. Back in Settings, click **Import existing records to Firestore** once. This writes all 736
   existing result records into your new project. Only do this once per project — running it
   again will create duplicates.

## 7. Everyday use
From here on, every browser that has this same Firebase config saved (via Settings → Connect)
and signs in with an admin account sees and edits the same live data. Add, edit, or remove a
result from a student's page; mark a student Active / Withdrawn / Graduated from the same page.

## 8. Deploy it somewhere real (optional)
The HTML file is fully self-contained — no build step. Either:
- **Firebase Hosting:** `firebase init hosting` (point the public directory at the folder
  containing this file, rename it to `index.html`), then `firebase deploy`.
- **Vercel:** same pattern you've used before — drag the folder into a new Vercel project, or
  `vercel deploy` from the CLI.

Once deployed, anyone visiting the URL still hits the sign-in screen and still needs a Firebase
Auth account you created — the app doesn't change behavior based on where it's hosted.

## Notes
- The Firebase config object is not a secret — it's meant to be visible in a web app's source,
  the same as every other Firebase project. Steps 3 and 4 (accounts + rules) are what actually
  keeps the data private, not hiding the config.
- If you ever want to restrict access to specific admin emails rather than "anyone with an
  account in this project," see the comment at the top of `firestore.rules` for the one-line change.
- PDF import (bulk-loading a CMS result slip) is not wired up yet — see the main chat for that.
