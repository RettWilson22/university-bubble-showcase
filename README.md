<p align="center">
  <img src="screenshots/icon.png" width="110" alt="University Bubble app icon" />
</p>

<h1 align="center">University Bubble</h1>

<p align="center">
  <b>A campus social app for Auburn University students: group chats, messaging, dining, housing, and events in one place.</b><br/>
  iOS · Android · Built solo with React Native, Expo, and Supabase
</p>

<p align="center">
  <img alt="Status" src="https://img.shields.io/badge/status-TestFlight%20beta-1B2567" />
  <img alt="Expo" src="https://img.shields.io/badge/Expo-SDK%2054-000020?logo=expo" />
  <img alt="React Native" src="https://img.shields.io/badge/React%20Native-0.81-61DAFB?logo=react&logoColor=black" />
  <img alt="TypeScript" src="https://img.shields.io/badge/TypeScript-5.9-3178C6?logo=typescript&logoColor=white" />
  <img alt="Supabase" src="https://img.shields.io/badge/Supabase-Postgres-3FCF8E?logo=supabase&logoColor=white" />
</p>

> **About this repo:** University Bubble is a private product, so its source code lives in a private repository. This page shows what the app does, how it's built, and the engineering problems I solved along the way.

## Screenshots

<p align="center">
  <img src="screenshots/login.jpg" width="200" alt="Login screen" />
  <img src="screenshots/dining.jpg" width="200" alt="Dining tab" />
  <img src="screenshots/restaurant.jpg" width="200" alt="Restaurant detail" />
  <img src="screenshots/campus.jpg" width="200" alt="Campus housing" />
</p>

## What it does

| Feature | Details |
|---|---|
| **Bubbles (group chats)** | Class and interest-based group chats with photos, polls, pinned messages, and a member roster you can see before joining. Restricted to verified `@auburn.edu` accounts. |
| **Direct messages** | Real-time 1:1 messaging with read tracking, exact unread badges, mute, and push notifications. |
| **Dining** | 350+ on- and off-campus restaurants with live open/closed status, hours, filters ("Open Late", "Budget Friendly"…), favorites, reviews, menus, and directions. |
| **Campus life** | ~70 off-campus apartments by category, a housing guide, campus events, and a study-spot guide. |
| **32 languages** | The full UI is translated, with language packs lazy-loaded to keep startup fast. |
| **Admin console** | In-app tools to edit restaurant and housing content live, moderate group chats, and ban users. No redeploy needed. |
| **Guest mode** | Browse dining and campus content without an account. |

## Tech stack

**Mobile:** React Native 0.81 · Expo SDK 54 · Expo Router (file-based navigation) · TypeScript · Reanimated · SF Symbols on iOS / Material Icons on Android · Expo Notifications · Expo Updates (over-the-air releases)

**Backend:** Supabase: PostgreSQL with Row-Level Security · Realtime (Postgres changes + broadcast) · Storage · Auth · Deno Edge Functions

**Tooling:** EAS Build & Submit (TestFlight) · Jest (`jest-expo`) · ESLint · SQL migrations in version control

```
┌──────────────────────────┐        ┌──────────────────────────────────────┐
│  iOS / Android app       │        │  Supabase                            │
│  React Native + Expo     │◄──────►│  Postgres + Row-Level Security       │
│                          │  RPC   │  28 SQL migrations, triggers, RPCs   │
│  Expo Router screens     │  REST  │                                      │
│  Context providers       │        │  Realtime: messages, unread, content │
│  Offline cache           │◄───────│                                      │
│  (stale-while-revalidate)│ socket │  Edge Functions (Deno):              │
└────────────┬─────────────┘        │   • email verification codes         │
             │                      │   • password reset                   │
     Expo Push / OTA updates        │   • push notification fan-out        │
                                    └──────────────────────────────────────┘
```

## Engineering highlights

**Security enforced on the server, not just the client**
- Email-based two-factor verification on login. Codes come from a cryptographically secure generator, are stored only as SHA-256 hashes, and expire. Password-reset codes work the same way and are verified with constant-time comparison.
- Every table is protected by Postgres Row-Level Security. Rules like "only Auburn students can post in group chats" and "banned users can't send messages" are enforced by **database triggers**, so a modified client can't get around them.
- Server-side rate limiting on logins, sign-ups, messages, and reviews (hourly and daily caps).
- Admin roles are granted by a server-side allowlist. The role column can't be written by clients.
- Input sanitization, length limits, and magic-byte checks on image uploads.

**Real-time messaging that stays correct**
- Diagnosed messages that weren't arriving live, and pushes that went to accounts users had signed out of. Root causes were an unmanaged realtime publication and a push-token table that let one device belong to many users. Fixed both with version-controlled migrations and a "last sign-in wins" token model.
- Exact unread counts computed server-side, with clock-skew-safe read markers. Messages stay ordered correctly across optimistic sends, realtime events, and pagination.
- Push notifications are sent by database triggers to an edge function that re-reads the message from the database. That makes notification content forge-proof and blocks replays.

**Performance**
- Cut the direct-message list from 5 network round trips to 3 by moving joins into a single Postgres function.
- Stale-while-revalidate caching so screens render instantly on launch.
- Lazy-loaded translation packs took about 267 KB off the startup bundle.

**Shipping and quality**
- Built and shipped solo to TestFlight across 20+ builds. JavaScript-only fixes ship over the air with no App Store review.
- 92 unit tests, including a custom guardrail test that scans every screen and fails the build if an admin-editable field is rendered without live-update support. That bug had shipped three times before the test existed.
- Ran multi-pass code audits that caught and fixed real issues before release (for example, poll voting that was blocked by a database policy, and a member count that never updated).

## What I learned
- **Design the security in the database.** Client checks are for user experience. Triggers and RLS are what actually protect the data.
- **Realtime systems fail quietly.** A subscription can look connected and still deliver nothing. Version-control *everything* the app depends on, including Supabase dashboard settings.
- **Write the test for the bug that keeps coming back.** A 50-line guardrail test ended a whole category of regressions.

<p align="center">
  Built by <b>Rett Wilson</b> · Auburn University · <a href="https://github.com/RettWilson22">@RettWilson22</a>
</p>
