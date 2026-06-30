# content-calendly
# FounderFlow

A lightweight content planning workspace for founders and their teams. Capture ideas, schedule posts on a calendar, manage brand kits, track goals, and run weekly reviews — all in one place, with a simple team/client permission model.

Built as a single static `index.html` file backed by Firebase (Authentication + Firestore). No build step, no framework, no backend server required.

## Features

- **Dashboard** — weekly snapshot of what's scheduled and what's still unscheduled
- **Ideas** — capture content ideas with notes, thumbnails, Drive links, and brand tags
- **Calendar** — drag-and-drop weekly scheduling
- **Brand Kit** — store brand colors, logo, and social media manager per brand
- **Goals** — track progress toward brand-specific targets
- **Weekly Review** — log wins, blockers, and focus areas each week
- **Roles** — two access levels:
  - **Team** — full create/edit/delete access across all brands, ideas, goals, and reviews
  - **Client** — read-only access, scoped only to the brand(s) they've been invited to
- **Invites** — team members can invite clients by email; access is granted instantly if the person already has an account, or queued as a pending invite until they register

## Tech stack

- Plain HTML/CSS/JavaScript (ES modules), no bundler
- [Firebase Authentication](https://firebase.google.com/docs/auth) — Email/Password sign-in
- [Cloud Firestore](https://firebase.google.com/docs/firestore) — data storage with real-time listeners
- Firestore Security Rules — enforce the team/client permission model server-side

## Project structure

```
.
├── index.html          # the entire app (landing page, login, app shell, logic)
└── firestore.rules     # Firestore security rules
```

## Setup

### 1. Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com) → **Create a project**
2. Enable **Authentication → Sign-in method → Email/Password**
3. Enable **Firestore Database** (start in production mode, pick a nearby region)

### 2. Add your Firebase config

In `index.html`, find the `firebaseConfig` object near the top of the `<script type="module">` block and replace the placeholder values with your project's config (Firebase Console → Project Settings → Your apps → SDK setup):

```js
const firebaseConfig = {
  apiKey: "YOUR_API_KEY",
  authDomain: "YOUR_PROJECT_ID.firebaseapp.com",
  projectId: "YOUR_PROJECT_ID",
  storageBucket: "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId: "YOUR_APP_ID"
};
```

> This config is safe to commit — it identifies your Firebase project but isn't a secret. Actual access control is enforced by Firestore Security Rules, not by hiding this object.

### 3. Deploy the Firestore rules

1. Firebase Console → **Firestore Database → Rules**
2. Paste in the contents of `firestore.rules`
3. Click **Publish**

### 4. Create your first user and set yourself as `team`

Since there's no public sign-up form (accounts are created by an admin), you'll need to:

1. Firebase Console → **Authentication → Users → Add user** — create your own email/password
2. Open `index.html` in a browser and log in with that account — this creates a `users/{uid}` document in Firestore with `role: "client"` by default
3. Firebase Console → **Firestore Database → Data → `users` collection** → open your document → edit the `role` field → change `client` to `team`
4. Refresh the app — you'll now see the full team workspace

Repeat step 3–4 for any other internal team members after they log in for the first time.

## How roles work

```
New user logs in  → role: "client" (default)
Team manually sets role: "team" in Firestore console

Team member   → sees all brands, ideas, goals, reviews, settings
              → can create/edit/delete everything
              → can manage client access per brand

Client        → sees only brands they've been granted access to
              → read-only (no create/edit/delete)
              → no Brand Kit editing, Goals, Weekly Review, or Settings pages
```

### Inviting a client

1. A team member clicks **Manage Access** on a brand card (Brand Kit page)
2. Enters the client's email
   - If that email is already registered → access is granted instantly
   - If not → a pending invite is stored and applied automatically the first time they log in
3. Access can be revoked, and pending invites cancelled, at any time from the same modal

## Deployment

This is a static site — deploy it anywhere that serves static files (Vercel, Netlify, GitHub Pages, Firebase Hosting, etc.).

**Vercel:**
1. Push `index.html` and `firestore.rules` to a GitHub repo
2. Import the repo into [vercel.com](https://vercel.com)
3. No build configuration needed — Vercel auto-detects and serves the static `index.html`

## Notes

- All data is stored in Firestore; there is no backend server.
- Brand logos and idea thumbnails uploaded through the UI are stored as base64 data URIs directly in Firestore documents (no Cloud Storage dependency), so keep uploaded images small (~1.5MB limit enforced in the UI).
- Local UI preferences (not app data) are cached in `localStorage` under the `founderflow.ui` key.