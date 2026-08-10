# Going live: connecting Stripe to your booking portal

The `book-direct.html` page currently **simulates** payment so you can see the whole flow. To take real money and hold a real deposit, you need a small server-side component — Stripe (correctly) will not let a plain web page charge cards directly. This guide gives you exactly what to deploy.

> **One-line summary:** the front end collects the booking; a tiny backend creates the Stripe charge and the deposit hold; Stripe handles the card data so you never touch it (this is what keeps you PCI-compliant).

---

## What you'll need

1. A **Stripe account** (free to create) → https://stripe.com
2. Your two API keys from the Stripe Dashboard → Developers → API keys:
   - **Publishable key** (`pk_live_…`) — safe to put in the web page
   - **Secret key** (`sk_live_…`) — **must stay on the server only, never in the HTML**
3. Somewhere to run a few lines of server code. Easiest options:
   - **Vercel** or **Netlify Functions** (free tier, no server to manage)
   - A small **Node/Express** app on any host

---

## How the two charges work

| What | Stripe mechanism | Behaviour |
|------|------------------|-----------|
| Licence fee (rent + cleaning) | **PaymentIntent**, captured immediately | Money taken now |
| Refundable damage deposit | **PaymentIntent** with `capture_method: 'manual'` | Card is *authorised* (held) but not charged. Release it after checkout, or capture part of it if there's damage. |

A manual-capture authorisation typically holds for up to 7 days. For month-long stays, the common pattern is to **not** pre-auth at booking, and instead either (a) take a separate deposit authorisation a few days before checkout, or (b) use a third-party deposit/vetting service such as **Superhog** or **Know Your Guest**, which is purpose-built for this and also does guest ID checks. For a serviced-accommodation business this is usually the cleaner route — worth pricing up.

---

## Step 1 — Backend endpoint (Node / serverless)

Create a serverless function (e.g. `api/create-payment.js` on Vercel):

```js
// api/create-payment.js
const stripe = require('stripe')(process.env.STRIPE_SECRET_KEY); // sk_live_… in env var, NOT in code

module.exports = async (req, res) => {
  if (req.method !== 'POST') return res.status(405).end();

  try {
    const { nights, nightlyRate, cleaningFee, email, name } = req.body;

    // Recalculate the price on the SERVER — never trust the amount sent by the browser.
    const amount = (nights * nightlyRate + cleaningFee) * 100; // pence

    const paymentIntent = await stripe.paymentIntents.create({
      amount,
      currency: 'gbp',
      receipt_email: email,
      description: `Plymouth serviced stay — ${nights} nights`,
      metadata: { guest: name }
    });

    res.json({ clientSecret: paymentIntent.client_secret });
  } catch (e) {
    res.status(500).json({ error: e.message });
  }
};
```

Set `STRIPE_SECRET_KEY` as an **environment variable** in your host's dashboard. Never paste it into the HTML.

---

## Step 2 — Front-end: swap the simulated `pay()` for Stripe

In `book-direct.html`, add Stripe.js in the `<head>`:

```html
<script src="https://js.stripe.com/v3/"></script>
```

Replace the contents of the card box and the `pay()` function with Stripe Elements (Stripe renders the secure card field — you don't build it):

```js
const stripe = Stripe('pk_live_YOUR_PUBLISHABLE_KEY'); // publishable key is fine here
const elements = stripe.elements();
const cardElement = elements.create('card');
cardElement.mount('#card-element'); // put <div id="card-element"></div> in the card box

async function pay() {
  const btn = document.getElementById('pay-btn');
  btn.disabled = true; btn.textContent = 'Processing…';

  // 1) ask your backend to create the PaymentIntent
  const res = await fetch('/api/create-payment', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({
      nights: currentNights,            // from your recalc()
      nightlyRate: NIGHTLY_RATE,
      cleaningFee: CLEANING_FEE,
      email: document.getElementById('g-email').value,
      name:  document.getElementById('g-name').value
    })
  });
  const { clientSecret, error } = await res.json();
  if (error) { alert(error); btn.disabled = false; btn.textContent = 'Pay & confirm booking'; return; }

  // 2) confirm the card payment
  const result = await stripe.confirmCardPayment(clientSecret, {
    payment_method: { card: cardElement,
      billing_details: { name: document.getElementById('cc-name').value } }
  });

  if (result.error) {
    alert(result.error.message);
    btn.disabled = false; btn.textContent = 'Pay & confirm booking';
  } else if (result.paymentIntent.status === 'succeeded') {
    go(5); // your existing success screen
  }
}
```

That's the whole change. The four-step flow, vetting checks, and licence acceptance you already have stay exactly the same — only the final charge becomes real.

---

## Step 3 — Test before going live

Stripe gives you **test keys** (`pk_test_…` / `sk_test_…`). Use them first.
Test card: `4242 4242 4242 4242`, any future expiry, any CVC. No real money moves.
When it all works, swap in the `live` keys.

---

## Step 4 — Things to wire up around it (don't skip)

- **Confirmation email + licence**: on success, email the guest their booking reference and the signed licence PDF. Stripe can send payment receipts; you handle the licence.
- **Calendar / double-booking**: the page doesn't know your availability. Block out booked dates — either keep a simple availability list, or keep your Airbnb calendar as the master and sync manually at first.
- **Webhooks**: add a Stripe webhook (`payment_intent.succeeded`) so your records update even if the guest closes the tab after paying.
- **Refunds / deposit release**: from the Stripe Dashboard you can refund or, for a manual-capture hold, release or partially capture it.

---

## The legal + insurance checklist (read before first live booking)

Taking bookings directly means you take on what Airbnb was doing for you:

- [ ] **Solicitor review** of the occupation licence (`Serviced-Accommodation-Licence-DRAFT.docx`). The document is a draft only.
- [ ] **Guest vetting / ID** — Airbnb verified guests for you. Use Superhog, Know Your Guest, or a manual ID + check process.
- [ ] **Damage protection** — no more AirCover. A deposit hold helps; a vetting service with damage cover is stronger.
- [ ] **Insurance** — standard home insurance won't cover paying occupiers. You need serviced-accommodation / short-let cover with public liability.
- [ ] **Tax** — direct income is still declarable. Keep records; check whether the £7,500 Rent-a-Room scheme or furnished-holiday-let rules apply to your situation.
- [ ] **Reality matches the paperwork** — actually provide the cleaning/linen/services and actually retain access. That's what keeps it a licence rather than a tenancy.
- [ ] **Data protection** — you're now holding guest personal data directly; handle it per UK GDPR.

None of the above is legal advice — it's a checklist of what to take advice on. One session with a property solicitor and a quick call with a short-let insurance broker covers most of it.
