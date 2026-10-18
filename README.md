# How to Host Budget App on GitHub + Firebase

`index.html` is the whole app. GitHub Pages serves it; Firebase (free Spark plan) syncs your data between phone and laptop. Without Firebase it still works, saving to the browser only.

## 1. Create a Firebase project
1. Open <https://console.firebase.google.com> and click **Create a project**.
2. Name it (e.g. `budget-os`), **disable Google Analytics**, click **Create project**.

## 2. Enable Anonymous Auth
**Build → Authentication → Get started → Sign-in method → Anonymous → Enable → Save.**

## 3. Create the Firestore database
**Build → Firestore Database → Create database → Start in *Test mode* → choose a region near you → Enable.**

## 4. Set Firestore rules (personal use)
Open **Firestore Database → Rules**, replace everything with the block below, then **Publish**.
(Test mode expires after 30 days, so this step is required.)

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /budgets/{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

> **Security note:** anonymous sign-in is open to anyone, and the Firebase config is visible in the page source. What protects your data is the passcode: it decides the secret path your data lives under, and the rules only allow access to that exact path (collection listing is not possible). A 6-digit code has only a million combinations, so avoid obvious numbers (123456, your birth year) and don't share the link. It is fine against casual access but not against a determined attacker. For stronger protection, switch to Google sign-in and restrict the rules to your own uid.

## 5. Get your `firebaseConfig`
**Project settings (gear icon) → General → Your apps → Web (`</>`) → register the app (skip Hosting) → copy the `firebaseConfig` block.**

## 6. Put the app on GitHub Pages
1. Create a repository on <https://github.com> (e.g. `budget-app`).
2. **Add file → Upload files** → upload `index.html` → **Commit changes**.
3. **Settings → Pages → Build and deployment → Source: Deploy from a branch → Branch `main`, folder `/ (root)` → Save.**
4. Wait about a minute. Your link will be `https://USERNAME.github.io/budget-app/`.

## 7. Connect each device (passcode, no keys to copy)
The Firebase config is already built into `index.html` (a web config identifies the project; it is not a password). Your data is protected by a **sync passcode** you choose:
1. Open the app. The **Sync across your devices** screen appears. Type a 6-digit passcode (numbers only) and tap **Save & connect**.
2. Your data is stored under an unguessable path derived from the passcode, so only devices that know it can read or write it.
3. On your other devices, open the app and type the same passcode, **or** tap the status bar → **Copy link / Share…** and open that link on the phone. The link contains the passcode, so send it only to yourself.

To use your own Firebase project instead, tap **Advanced: use my own Firebase project** on that screen, or edit `firebaseConfig` near the top of the script.

## 8. Test the sync
1. Open the link on your laptop and on your phone.
2. Add an entry on one device. It appears on the other within a few seconds.
3. Turn on airplane mode, add an entry (the bar shows **Offline - Saved Locally**), turn it off, and watch it upload.

## Good to know
- **Status bar** (under the header): grey = local only, yellow = syncing, green = synced, red = offline. It also shows when it last synced.
- **First connect:** data already on a device is merged into the cloud (nothing is lost). After that, the cloud is the source of truth. Deleting or clearing on one device does so everywhere.
- **Where data lives:** `budgets/budget_app_main_v1/entries/{id}` and `budgets/budget_app_main_v1/metadata/config` (categories, tags, budgets). Change `APP_ID` in the script to keep separate datasets.
- **Offline:** your data is always kept in the browser and in Firestore's offline cache. Reopening the page itself with no network relies on your browser's cache. The Firebase library loads from Google's CDN, so if it can't load, the app runs in local-only mode and syncs next time.
- The weekly auto backup and the JSON/CSV export still work and stay local to each device.
- Free-plan limits (50k reads / 20k writes per day) are far above personal budgeting use.
