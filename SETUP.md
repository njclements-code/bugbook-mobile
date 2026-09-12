# Personal Bug Book — mobile (Android) setup

This is a static web app (a PWA — installable, works full-screen, no Play
Store needed). It talks to Google Drive directly, reading and writing the
exact same `bugbook-data.json` file your Mac's Bug Book app already syncs —
so entries you log on your phone show up on your Mac and vice versa.

Two one-time jobs: (1) host these files somewhere public, (2) create free
Google API credentials so the app is allowed to read/write that one file.
Neither needs to be repeated after this.

## Part A — Host the files (GitHub Pages)

1. Go to github.com and sign in (or create a free account).
2. Click **New repository**. Name it something like `bugbook-mobile`. Set it
   to **Public**. Don't add a README. Click **Create repository**.
3. On the empty repo page, click **uploading an existing file**.
4. Drag in every file from this folder, keeping the folder structure:
   `index.html`, `manifest.json`, `sw.js`, and the `icons/` folder with its
   4 PNGs. Commit the upload.
5. Go to **Settings → Pages** (left sidebar). Under "Build and deployment",
   set Source to **Deploy from a branch**, Branch to **main** and folder to
   **/ (root)**. Save.
6. Wait ~1 minute, then refresh that Settings → Pages screen — it will show
   your live URL, something like:
   `https://<your-github-username>.github.io/bugbook-mobile/`
   That's the address you'll open on your phone. Keep it handy — you'll need
   it in step A of Part B too.

## Part B — Google Cloud credentials

1. Open [Google Cloud Console](https://console.cloud.google.com/) signed in
   as **njclements@gmail.com** (the same account your Drive syncs).
2. Create a new project (top bar → New Project). Name it e.g. "Bug Book
   Mobile". Note the **Project number** shown on the project dashboard —
   you'll paste that into the app later (optional but recommended).
3. Enable the two APIs this app needs:
   - [Enable Google Drive API](https://console.cloud.google.com/apis/library/drive.googleapis.com)
   - [Enable Google Picker API](https://console.cloud.google.com/apis/library/picker.googleapis.com)
   (Click each link, make sure your new project is selected top-left, click Enable.)
4. Configure the consent screen: **APIs & Services → OAuth consent screen**.
   - User type: **External**.
   - Fill in app name ("Personal Bug Book"), your email as support + developer contact.
   - Scopes: you don't need to add any here — the app requests
     `drive.file` at sign-in time, which Google treats as a
     non-sensitive scope, so this app does **not** need Google's
     verification review.
   - Test users: add njclements@gmail.com.
   - Once created, go to the consent screen's summary and change
     **Publishing status** from "Testing" to **"In production"**. This
     matters: in Testing mode, Google expires your sign-in every 7 days
     and you'd have to redo Part B step 6 weekly. In Production (fine for
     an unverified app using only non-sensitive scopes like this one),
     it doesn't expire that way.
5. Create credentials — **APIs & Services → Credentials → Create
   Credentials**:
   - **API key**: create one. Optionally click into it and under
     "Application restrictions" choose "Websites" and add your GitHub
     Pages URL from Part A. Copy the key.
   - **OAuth client ID**: type **Web application**. Under "Authorized
     JavaScript origins" add your GitHub Pages origin, e.g.
     `https://<your-github-username>.github.io` (no trailing slash, no
     path). Save. Copy the Client ID.
6. On your phone, open your GitHub Pages URL in Chrome. You'll see the
   "Connect to Google Drive" screen:
   - Paste the **OAuth Client ID**, **API key**, and (optional) **project
     number** → Save.
   - Tap **Sign in with Google** → choose njclements@gmail.com → allow.
   - Tap **Choose file…** → in the picker, find and select the existing
     `bugbook-data.json` inside your **BugBook** folder in Drive (the same
     one `app.py`'s `DATA_DIR` points at). This links the app to that file
     — it will not create a new one.
7. You're in. From here, tap the browser menu → **Add to Home screen** (or
   **Install app**) so it opens full-screen like a normal app icon, no
   address bar.

## Notes and known limits

- **Sign-in per app open.** For security, the access token isn't kept
  across full app restarts — expect to tap "Sign in with Google" again
  each time you reopen the app after it's been closed a while (it stays
  signed in while you're actively using it). This is normal for this kind
  of no-backend setup.
- **Last write wins.** Same caution as the desktop app: if you edit on
  the phone and the Mac within the same moment, whichever saves last
  overwrites the other. Fine for one person logging from one device at a
  time.
- **Daily reminder is best-effort.** The in-app "Enable reminder
  notifications" button uses Chrome's Periodic Background Sync API. Chrome
  — not this app — decides how often it actually fires, based on how much
  you use the installed app; it is not a precise daily alarm like the Mac
  menu-bar version. Keep relying on the Mac notification as the reliable
  one.
- **Credentials live only on your phone.** The Client ID / API key / linked
  file ID are stored in this browser's local storage on your phone, never
  sent anywhere except Google's own APIs.
