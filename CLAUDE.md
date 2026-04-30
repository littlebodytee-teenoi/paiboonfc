# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

A real-time football player sign-up app (in Thai) backed by Firebase Firestore. No build system — the entire frontend is a single `index.html` with vanilla ES module JavaScript. Firebase SDK is loaded from CDN (`gstatic.com`).

## Setup

1. Copy `firebase-config.example.js` → `firebase-config.js` and fill in real credentials from Firebase Console.
2. Deploy Firestore security rules: `firebase deploy --only firestore:rules`
3. Serve `index.html` with any static file server, e.g.:
   ```
   npx serve .
   # or
   python3 -m http.server 8080
   ```

`firebase-config.js` is gitignored — never commit it.

## Architecture

All application logic lives inside a single `<script type="module">` block in `index.html`.

**Firestore collections:**
- `players` — `{ name: string, createdAt: Timestamp }`, ordered by `createdAt asc`
- `groups` — `{ colorIdx: number, playerIds: string[], createdAt: Timestamp }` — manual groupings that survive team randomization

**Data flow:** Both collections are subscribed to via `onSnapshot` listeners. Every change (local or remote) triggers a full `render()` call that rebuilds the player list DOM from the in-memory `players` and `groups` arrays.

**Team randomization logic:** Players/groups are treated as indivisible *units*. Units are shuffled and greedily assigned to three teams (red/yellow/blue) using a "fewest players first" strategy, keeping grouped players together.

**Writes always use `writeBatch`** when multiple documents must be updated atomically (e.g., deleting a player also cleans up any group containing them; creating a group removes players from their prior groups if they overlap).

## Firestore Security Rules

`firestore.rules` enforces field allowlists and basic type/size validation for both collections. After editing, redeploy with `firebase deploy --only firestore:rules`.
