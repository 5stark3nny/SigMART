# SigMART — Full-stack marketplace

**Good finds. Better living.**

SigMART is a full-stack online storefront with a responsive frontend and Node.js backend. It includes product search/filter/sort, favourites, cart, account registration/sign-in, order placement, order history, newsletter signups, stock management and a basic admin dashboard.

## Run locally

Requires Node.js 20+.

```bash
npm install
npm start
```

Open http://localhost:3000. The catalogue is seeded on first launch; data is stored in `data/store.json`.

## Deploy the full application

GitHub Pages hosts static files only and cannot run this backend. Deploy as a **Render Web Service** instead. The included `render.yaml` defines the Node service and health check.

1. In Render, choose **New → Blueprint** and select this repository (or create a Web Service with build command `npm install` and start command `npm start`).
2. Set `ADMIN_EMAIL` to the email address that should receive store-admin access.
3. Ensure `SESSION_SECRET` is a long random value (the Blueprint generates one).
4. Deploy and check `/api/health`.

## Features

- Responsive product catalogue with search, category filters and sorting
- Password hashing using Node scrypt and signed HttpOnly session cookies
- Cart and checkout; order creation validates stock and decrements inventory
- Customer order history
- Admin dashboard, product creation and order status updates
- Newsletter email collection

## Important production limits

- Checkout currently uses **cash on delivery**. No card data is collected and no online payments are processed. To accept online payments, integrate Paystack or Stripe using server-side secrets and verified webhooks.
- Newsletter addresses are stored but no emails are sent.
- The JSON file database is a starter store. Free hosting may reset local files after restart/redeploy; use PostgreSQL or another managed database for production.
- For a public production launch, add rate limiting, CSRF protection, account recovery, database backups and security review.

Admin access is granted only when the registering email matches the server's `ADMIN_EMAIL` environment variable. Set that value before registering the intended admin account.


## Admin order notifications

SigMART can send order alerts to the admin by email and Telegram. Notifications run server-side after a cash-on-delivery order is saved, after Paystack checkout is successfully initialized, and again when Paystack payment is verified as successful. A notification-provider failure is logged and does not cancel a valid order or payment.

Configure these environment variables on the Render web service:

- `ADMIN_EMAIL`: admin inbox (default fallback: `5stark3nny@gmail.com`).
- `RESEND_API_KEY`: API key from [Resend](https://resend.com/). Without it, email alerts are disabled.
- `EMAIL_FROM`: sender identity, e.g. `SigMART <orders@your-verified-domain.com>`. Verify the sending domain in Resend first. For initial testing, Resend's `onboarding@resend.dev` sender is restricted by Resend's test-mode rules.
- `TELEGRAM_BOT_TOKEN`: token from Telegram's official `@BotFather`.
- `TELEGRAM_CHAT_ID`: the private chat ID for the admin who starts the bot and sends it a message first.

To obtain the Telegram chat ID: create a bot with `@BotFather`, open the bot's private chat and press Start/send a message, then visit `https://api.telegram.org/botYOUR_BOT_TOKEN/getUpdates` and copy the numeric `message.chat.id`. Keep the bot token private. Add these values in Render → SigMART service → Environment; never commit real tokens to GitHub. Redeploy after changing environment variables.

Paystack notifications distinguish the initial order (payment initiated/pending) from the verified payment-confirmed alert. Existing Paystack checkout, webhook verification, PostgreSQL persistence, and order storage remain in place.
