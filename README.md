# Plymouth Contractor Stay — direct booking site

A two-page static site plus a request-to-book flow. **No Stripe / no payments** — guests send a booking request, you confirm availability and pricing, then take payment your own way. Payments can be added later (see the separate Stripe setup guide).

```
plymouth-site/
├── public/
│   ├── index.html        ← landing page
│   ├── book.html         ← request-to-book flow (4 steps + confirmation)
│   └── images/           ← put your photos here (see step 2)
├── vercel.json           ← deploy config
└── README.md
```

---

## Getting it live — 4 steps

### 1. Connect the booking form to your inbox  *(5 min)*
Right now the form works but sends nowhere (demo mode). Pick a free form service and paste your endpoint in:

- **Web3Forms** (https://web3forms.com) or **Formspree** (https://formspree.io) — both free, both give you a URL.
- Open `public/book.html`, find this line near the bottom:
  ```js
  const FORM_ENDPOINT = '';
  ```
  and paste your endpoint URL between the quotes. Requests will then land in your email.

*(Skip this and the form still works as a demo — it just won't email you.)*

### 2. Add your own photos  *(10 min — recommended)*
The site currently hotlinks Airbnb's images. Those can break over time, so host your own:
1. Download your listing photos.
Photos are already wired — just ensure your images folder (with the .JPG files) is inside public/. Missing photos show a placeholder.

*(Skip this and the site still displays — it just relies on Airbnb's image links.)*

### 3. Deploy to Vercel  *(5 min — same as your other projects)*
1. Put this folder in a new GitHub repo (via GitHub Desktop or the browser upload).
2. In Vercel → **Add New → Project → import the repo**.
3. Framework preset: **Other**. Root/output: leave default (it serves `public/`).
4. Deploy. You'll get a live `*.vercel.app` URL immediately.

### 4. Point your domain  *(15 min)*
1. Buy a domain (Namecheap / GoDaddy) — e.g. `plymouthcontractorstays.co.uk`.
2. In Vercel → Project → **Settings → Domains → add your domain**, then follow the DNS instructions.

Done — you're live and taking booking requests.

---

## When you're ready to take payment on-site
Switch on Stripe using the separate **stripe-backend-setup.md** guide. The booking flow is already built for it — you'd add the payment step back as stage 5 and deploy the small `api/` function. **Get the licence reviewed by a solicitor first.**

## Editing pricing shown in the summary
In `book.html`, near the top of the script:
```js
const NIGHTLY_RATE = 75;
const CLEANING_FEE = 60;
const DEPOSIT_HOLD = 300;
```
These drive the live price estimate on the request form. (Final pricing is whatever you confirm by email.)

## Before your first *paid* booking — checklist
- [ ] Solicitor has reviewed the occupation licence
- [x] Short-let insurance in place
- [ ] Guest vetting / ID process decided (manual, or Superhog / Know Your Guest)
- [ ] Availability kept in sync so you don't double-book with Airbnb
