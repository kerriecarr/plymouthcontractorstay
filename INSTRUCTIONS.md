# Plymouth Contractor Stay — everything you need, one page

Your booking form is **already wired to your email** (Web3Forms key baked in). Booking requests will arrive in the inbox tied to that key. Nothing else to configure for the form.

Below is the whole path from folder → live site. You're on a laptop, so this is the easy version.

---

## What's in this folder

```
plymouth-site/
├── public/
│   ├── index.html        landing page
│   ├── book.html         request-to-book flow (form already emails you)
│   ├── availability.html availability calendar (you update it manually)
│   └── images/           put your photos here
├── vercel.json           deploy config
├── README.md             short version
├── INSTRUCTIONS.md       ← this file
├── stripe-backend-setup.md               (for adding card payments later)
└── Serviced-Accommodation-Licence-DRAFT.docx   (solicitor review before paid bookings)
```

---

## Updating the availability calendar

The calendar is **manually controlled** — deliberately, so it doesn't inherit the far-ahead blocks you keep on Airbnb. It shows every date as open *except* the ones you list.

To mark dates as booked: open `public/availability.html`, find the block near the top of the `<script>` that starts:

```
let BOOKED = [
```

Add one line per taken period. `start` is the first night; `end` is the checkout day (the day it's free again):

```
let BOOKED = [
  { start: '2026-09-01', end: '2026-10-01' },   // September taken
  { start: '2026-12-20', end: '2027-01-05' },   // held over Christmas
];
```

Delete a line when a period frees up. Commit the change (or re-upload the file) and Vercel redeploys automatically. That's the whole system — no iCal, no sync delay, no surprises.

---

## STEP 1 — Your photos (already set up)

The site is already wired to use local photos from `public/images/`. Just make sure your `images` folder (with your `.JPG` files) is inside `public/`. The expected filenames are:

- `living.JPG` (main living area — hero image)
- `kitchen.JPG`
- `bedroom.JPG`
- `bedroom2.JPG`
- `garden.JPG`
- `bathroom.JPG`
- `exterior.JPG`
- `host.jpg` (Kerrie's profile photo — lowercase)

No script to run — the pages already point at these names. If any photo is missing (e.g. you haven't added `host.jpg` yet), the site shows a tidy navy placeholder in its place, so nothing looks broken. Add the missing one whenever.

---

## STEP 2 — Put the files on GitHub  (~5 min)

Use whichever you prefer:

**GitHub Desktop (easiest):** New repository → drag the `plymouth-site` contents in → Commit → Publish.

**Or github.com in the browser:** New repo (Public) → "uploading an existing file" → drag the whole folder in → Commit.

Name the repo something like `plymouth-stay`.

---

## STEP 3 — Deploy to Vercel  (~5 min — same as Stayflow / AKWA)

1. Go to vercel.com → sign in with GitHub.
2. **Add New → Project** → import `plymouth-stay`.
3. Framework preset: **Other**. Leave the rest default (it serves the `public/` folder).
4. **Deploy.** You get a live `*.vercel.app` link in about a minute.

Open the link, run through a test booking request, and check it lands in your email.

---

## STEP 4 — Connect your domain (plymouthcontractorstays.co.uk)  (~15 min)

You've already bought `plymouthcontractorstays.co.uk` — now point it at Vercel.

1. Vercel → your project → **Settings → Domains → Add**.
2. Add **both**: `plymouthcontractorstays.co.uk` and `www.plymouthcontractorstays.co.uk`. Vercel will suggest making one redirect to the other — using `www` as the primary (to match your branding) is fine; let the bare domain redirect to it.
3. Vercel shows you DNS records to add. Log in to wherever you bought the domain, find its **DNS settings**, and add them:
   - Typically an **A record** for the bare domain pointing to Vercel's IP (`76.76.21.21`), and
   - a **CNAME record** for `www` pointing to `cname.vercel-dns.com`.
   - Vercel shows the exact values — always use what Vercel displays, not these examples, in case they change.
4. Save. DNS can take anywhere from a few minutes to a couple of hours to propagate. Vercel's Domains page shows a green check when it's live, and issues the HTTPS certificate automatically.

Once it's green, `https://www.plymouthcontractorstays.co.uk` is your live site.

---

## Editing the pricing shown on the form

In `public/book.html`, near the top of the `<script>`:
```
const NIGHTLY_RATE = 75;
const CLEANING_FEE = 60;
const DEPOSIT_HOLD = 300;
```
These drive the live estimate on the request form. Final pricing is whatever you confirm by email.

---

## How the booking flow works now

1. Guest fills in details → confirms quick checks → accepts the occupation licence → sends request.
2. **You get an email** with all their details (dates, guests, project, individual-or-company).
3. You reply with availability + exact price, then take payment your own way.

No card payment happens on the site yet — that's deliberate, and lower-risk, until the licence is solicitor-reviewed.

---

## Before your first PAID booking — checklist

- [ ] **Solicitor** has reviewed `Serviced-Accommodation-Licence-DRAFT.docx`
- [x] Short-let **insurance** in place
- [ ] **Vetting / ID** process decided (manual, or Superhog / Know Your Guest)
- [ ] **Availability** kept in sync with Airbnb so you don't double-book

When those are sorted and you want card payments on-site, open `stripe-backend-setup.md` — the booking flow is already built to slot Stripe in.

---

## Test the form right now (optional)

Before deploying, open `public/book.html` directly in your browser, fill it in, and send. Because Web3Forms is a hosted endpoint, the email will send even from your local file — a quick way to confirm it reaches your inbox before going live.
