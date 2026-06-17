---
name: Firebase video limitation
description: TNC Firebase-secured videos cannot be played on the web; _fs_id is a UUID with no public access
---

## The Problem

TNC Nursing app stores ~137 video sessions as Firebase secured content. The CRM field `_fs_id` (e.g. `ae378113-7e76-4444-9e33-b1b27ec75239`) is a UUID that maps to a Firebase Realtime Database or Firebase Storage entry requiring Firebase SDK auth.

Probing common Firebase Storage bucket patterns returns 404 for all:
- `tncnursing.appspot.com`
- `in-tncnursing-app.appspot.com`
- `tnc-nursing.appspot.com`

These videos are auth-gated and cannot be streamed without Firebase service account credentials.

## Current Implementation

- `parseChapter` in proxy.ts: detects `_fs_id` presence → sets `contentType: "firebase"`, includes `firebaseId: string | null`
- `watch.tsx`: shows `SecuredVideoCard` (dark UI with shield icon, explains migration is in progress)
- `course-detail.tsx`: firebase sessions show as clickable links labeled "SECURED" (amber), NOT "Download App"
- No "Download App" / Play Store links anywhere on the site

**Why:** Firebase Storage access requires a valid Firebase ID token. Without the app's Firebase config/service account, the videos are inaccessible from the web server.

**How to apply:** If TNC ever provides their Firebase service account JSON, add it as an env secret and use the Firebase Admin SDK to generate signed download URLs server-side in a new `/api/firebase-video/:fsId` route.
