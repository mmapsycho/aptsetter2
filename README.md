# Dr. Marina Farrag — Dental Clinic Website

A single-file website (`index.html`) with a live editor and booking system, backed by your **avr-dental** Firebase project. No build step — deploy the file as-is to GitHub Pages.

## What's in it

- **Public site** — hero, about, services, gallery, reviews, a 4-step booking wizard (service → date → time → contact details), footer.
- **Staff Login** (top-right corner) — signs in with a Firebase Auth account you create yourself (see below). No public sign-up exists anywhere on the site.
- **Edit site** (visible once signed in) — a panel to edit everything: colors, logo, hero slides, about text, services, gallery photos, reviews, contact info, socials, nav, booking hours, footer. Saves straight to Firestore and updates the live site immediately for every visitor.
- **Bookings** (visible once signed in) — a live list of every booking request, with a status dropdown (new / confirmed / completed / cancelled) and delete button.
- **Image uploads** — every image field in the editor has an "Upload" button that sends the file to Firebase Storage and fills in the URL automatically. You can also just paste an image URL instead.

## One-time Firebase setup (required before this works)

The site already has your Firebase project's config wired in. Three things need to be set up **in the Firebase Console** before login, editing, and bookings will actually work — these can't be done from the website itself, for security reasons.

### 1. Create staff accounts
Firebase Console → your **avr-dental** project → **Authentication** → **Users** → **Add user**. Enter an email and password for each staff member who should be able to log in and edit the site. There's no public sign-up page — accounts can only be created here.

If "Email/Password" sign-in isn't already enabled: **Authentication → Sign-in method → Email/Password → Enable**.

### 2. Firestore database + rules
If you haven't already, **Firestore Database → Create database** (production mode is fine — the rules below handle access).

Then go to **Firestore Database → Rules** and replace the contents with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Site content: anyone can read (so the site renders), only signed-in staff can edit
    match /site/{docId} {
      allow read: if true;
      allow write: if request.auth != null;
    }

    // Bookings: anyone can submit a request, only signed-in staff can read/manage them
    match /bookings/{bookingId} {
      allow create: if true;
      allow read, update, delete: if request.auth != null;
    }
  }
}
```

Click **Publish**.

### 3. Storage rules (for the image upload buttons)
**Storage** → if you haven't already, click **Get started** to create the default bucket. Then go to **Storage → Rules** and use:

```
rules_version = '2';
service firebase.storage {
  match /b/{bucket}/o {
    match /{allPaths=**} {
      allow read: if true;
      allow write: if request.auth != null;
    }
  }
}
```

Click **Publish**.

That's it — with these three steps done, staff can sign in, edit any part of the site, upload photos, and manage bookings from the live page.

## How editing works day-to-day

1. Open the live site, click **Staff Login** (top right), sign in with a staff account.
2. Click **Edit site** to open the editor — it's organized into collapsible sections (Brand & colors, Hero slides, Services, Gallery, Reviews, Booking hours, etc.). Add/remove items with the buttons inside each section, upload or paste image URLs, then **Save changes**. Every visitor sees the update immediately — no redeploy needed.
3. Click **Bookings** to see every request patients have submitted, change its status, or delete it.
4. **Sign out** when done.

Anything not covered by the editor (fonts, layout, page structure) still lives in the `index.html` file itself and needs a code edit + redeploy to change.

## Deploying to GitHub Pages

1. Create a new repository on GitHub (e.g. `marina-farrag-site`).
2. Upload `index.html` to the repository — either via the GitHub web UI ("Add file → Upload files") or:
   ```bash
   git init
   git add .
   git commit -m "Initial site"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo-name>.git
   git push -u origin main
   ```
3. On GitHub, go to **Settings → Pages**.
4. Under "Build and deployment", set **Source** to `Deploy from a branch`, branch `main`, folder `/ (root)`, then **Save**.
5. GitHub will publish the site at `https://<your-username>.github.io/<repo-name>/` within a minute or two.

### Important: add your domain to Firebase's allowed list
Firebase Auth only allows sign-in from domains you've approved. Once your GitHub Pages URL (or custom domain) is live: **Firebase Console → Authentication → Settings → Authorized domains → Add domain**, and add your `github.io` URL (and your custom domain later, if you set one up).

To use your own domain (e.g. `dr-marinafarrag.com`) instead of the github.io address, add a `CNAME` file containing just your domain name to the repository root, point your domain's DNS to GitHub Pages per GitHub's custom-domain docs, and add that domain to Firebase's authorized domains list too.

## A note on the Firebase config in the code
The `firebaseConfig` object (apiKey, project id, etc.) embedded in `index.html` is meant to be public — it identifies your project to Firebase, it isn't a secret. All real access control happens through the Firestore/Storage rules above and the staff accounts in Authentication, so keep those locked down rather than trying to hide the config.
