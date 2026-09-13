# MoLab Cerebral Garden

A gesture-controlled "garden" art experience (hand-tracked, camera never shown on screen) gated
behind a **Rs. 110 / 3-minute-session** payment, built the same way as the Paratha·Shawarma·Lassi
reference app you shared: a single Node/Express server that hosts the game, a staff admin panel
for swapping the payment QR, and a payment-gate API for confirming payment.

## What changed from the Cerebral Garden HTML you uploaded

- Nothing about the garden itself (gestures, drawing, flowers, help/feedback modals, auto-download)
  was altered — it's the same code, just wrapped behind a payment gate.
- Added: start screen, payment screen (name → QR → ref code → confirmation), staff-override modal,
  and a countdown timer pill (`⏱ 3:00`) that now displays during the session and hard-stops the
  camera + garden when it hits zero.
- Branding: **MoLab Cerebral Garden**, your MoLab logo as the badge and favicon, and a color palette
  that blends the garden's existing cyan/gold/rose/violet accents with your logo's navy/teal.
- Price: Rs. 110 per 3-minute session (one session per payment — no "2 free sessions" carry-over
  like the reference app had).

## ⚠️ Before you take real payments

- **No real payment QR is included.** `public/qr-current.jpg` ships as a clearly-labelled
  placeholder ("QR NOT SET"). I did not carry over the QR code from your reference zip, because
  that QR pays into a specific person's real bank account — reusing it here would route your
  customers' money to them, not to you. Upload your **own** Raast/JazzCash/Easypaisa receiving QR
  through `/admin.html` first.
- **This is a real payment-confirmation backend, not a payment processor.** It doesn't move money
  itself — it watches for a bank/wallet notification (via the same kind of Android
  SMS/notification-listener + webhook pattern as your reference app) and matches it to a pending
  session by payer name. You still need your own bank notification automation (e.g. MacroDroid/
  Tasker) pointed at `/api/mcb-webhook` with your `WEBHOOK_SECRET`.
- Change `ADMIN_PASSWORD` and `WEBHOOK_SECRET` (env vars) and `STAFF_OVERRIDE_PIN` (top of the
  `<script>` in `public/index.html`) before going live — all three ship with placeholder defaults.
- The expected amount strings in `server.js` (`EXPECTED_AMOUNT_STRINGS`) assume your bank's
  notification text contains "110" or "Rs. 110" — check your actual bank's wording and adjust if
  needed.

## Local testing

```
npm install
ADMIN_PASSWORD=test123 WEBHOOK_SECRET=test-secret node server.js
```

Then open `http://localhost:4000` for the app and `http://localhost:4000/admin.html` for the admin
panel. Since there's no real QR/bank hooked up yet, use **Staff Override** (PIN `2468` by default —
change it) on the payment screen to skip straight to a session while testing.

## Deploying (e.g. on Render)

1. Push this folder to a GitHub repo.
2. Render → **New → Web Service** → connect the repo.
3. Build command: `npm install` · Start command: `node server.js`.
4. Add env vars: `ADMIN_PASSWORD`, `WEBHOOK_SECRET`.
5. Deploy, then visit `/admin.html` to upload your real payment QR.
6. Point your bank-notification automation's webhook action at
   `https://<your-app>.onrender.com/api/mcb-webhook`.

## Important: ephemeral filesystem

Most free hosting tiers wipe on-disk changes (like an uploaded QR) on every redeploy or restart.
Re-upload the QR after any redeploy, or add a persistent disk if your host supports one.

## Files

- `server.js` — Express app: static hosting + admin auth/upload + payment session API
- `public/index.html` — the app: start screen → payment gate → Cerebral Garden (with session timer)
- `public/admin.html` — staff QR-upload + stats panel
- `public/logo.png`, `public/favicon.png` — your MoLab logo
- `public/qr-current.jpg` — **placeholder** QR graphic (replace via admin panel)
- `data/qr-meta.json` — stores the current QR's filename + expiry date
