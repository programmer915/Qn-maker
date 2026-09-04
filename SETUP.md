# Setup

The app is a static site, but the login/cloud-storage features need a
Firebase project that **you** create and own (it's free on Firebase's
"Spark" plan for this app's usage). These steps only need doing once.

## 1. Create a Firebase project

1. Go to https://console.firebase.google.com and click **Add project**.
2. Give it a name (e.g. "paper-trail"), finish the wizard.

## 2. Register a Web App and get your config

1. In the project's console, click the **</>** (Web) icon to add a web app.
2. Give it a nickname — you don't need Firebase Hosting set up through this
   wizard if you're hosting elsewhere (e.g. GitHub Pages), but it's fine to
   enable it too.
3. Firebase will show you a `firebaseConfig` object like:
   ```js
   const firebaseConfig = {
     apiKey: "AIza...",
     authDomain: "paper-trail-xxxxx.firebaseapp.com",
     projectId: "paper-trail-xxxxx",
     storageBucket: "paper-trail-xxxxx.appspot.com",
     messagingSenderId: "...",
     appId: "..."
   };
   ```
4. Open `index.html`, find the `firebaseConfig` placeholder near the top of
   the `<script type="module">` block, and paste your real values in.
   (This config is not a secret — it's meant to be public in client code;
   access is controlled by the security rules in step 5, not by hiding it.)

## 3. Turn on Authentication

1. In the console, go to **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Email/Password**.

## 4. Turn on Firestore

1. Go to **Build → Firestore Database → Create database**.
2. Choose **production mode** (the rules in this repo lock it down
   correctly) and pick a region close to you.

(There's no Storage step here — this version doesn't use Firebase
Storage, since Google now requires a paid Blaze plan to use it at all.
Images work fully while your tab is open but aren't saved to the cloud;
see "Known limits" in `README.md`.)

## 5. Deploy the security rules

The rules in `firestore.rules` restrict every teacher to their own
data — deploy them so they actually take effect (until you do, Firestore
defaults to locked-down and nothing will save). Easiest way — no
terminal needed: open **Firestore Database → Rules** in the console,
paste in the contents of `firestore.rules`, and click **Publish**.

If you'd rather use the CLI instead:
```bash
npm install -g firebase-tools     # one-time, if you don't have it
firebase login
firebase use --add                # pick your project, e.g. paper-trail-xxxxx
firebase deploy --only firestore:rules
```

## 6. Host the site

You can host `index.html` anywhere static (GitHub Pages, Firebase
Hosting, Netlify, ...) — it doesn't need a server. If you want to use
Firebase Hosting itself:

```bash
firebase deploy --only hosting
```

If you're hosting elsewhere (e.g. GitHub Pages), just upload `index.html`
— the `firebase.json`/`firestore.rules`/`package.json` files here are
only used by the `firebase` CLI for the rules/hosting deploy above, not
by the browser.

## 7. Try it

Open the site, tap **Sign up**, create an account with an email +
password. You should land in the app; anything you upload/add/generate
now saves to your account and will still be there next time you sign in
from any device (except images — see "Known limits" in `README.md`).

## Troubleshooting

- **"Missing or insufficient permissions" on save** — the rules from
  step 5 haven't been published/deployed yet, or you're not signed in.
- **OCR silently does nothing / console shows a 404 for `pdf.worker.min.js`**
  — the pdf.js CDN build in `index.html` changed or is unreachable; check
  the browser console for the exact failing URL and swap in a current one
  from https://www.jsdelivr.com/package/npm/pdfjs-dist.
- **A very large question bank fails to save** — Firestore documents cap
  out at 1MB; see "Known limits" in `README.md`.
