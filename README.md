# thefoods

A mobile-friendly, installable ordering website with a small Node.js backend. It uses only Node built-ins and stores its database in `data/store.json`; no build step or database server is needed for a local demo. Customer accounts, owner sessions, menu data and order history are shared by everyone visiting the same deployment.

## Run the no-charge demo

1. Install Node.js 20 or newer on Windows.
2. Open PowerShell in this folder and run `Copy-Item .env.example .env`.
3. Edit `.env`: set `OWNER_EMAIL` and a long `OWNER_PASSWORD` (this is the one-time setup key), keep `DEMO_MODE=true`, and replace `SESSION_SECRET` with a random secret.
4. Run `node server.js` (or `npm start` if npm is installed), then open `http://localhost:3000` on your computer. To try it on a phone on the same Wi-Fi, open `http://<computer-local-IP>:3000`.
5. Place pickup and delivery orders. The demo UPI option marks a test payment as `demo_paid`; it never opens a payment screen or charges a card/account. Cash can also be tested. Demo account verification displays a one-time code on screen and in the server terminal.
6. Choose **Owner sign in**. On first use, submit your email, the `OWNER_PASSWORD` from `.env` as the setup key, then choose a new owner password of at least 14 characters. After setup, the owner login uses that new password.

Use HTTPS behind a maintained reverse proxy or hosting platform before sharing publicly. Set `NODE_ENV=production`, `DEMO_MODE=false`, `BASE_URL` to the HTTPS site, a long `SESSION_SECRET`, and strong owner credentials. Restrict network access to the app server, back up `data/store.json`, and keep `.env` out of source control. This simple JSON database is suitable for a single-instance starter; use a managed database and session store before scaling to multiple server instances.

## Before taking real orders

Configure all of these in the owner dashboard and hosting environment:

- **Business:** verified shop name, contact phone/email, full shop address, opening hours, menu names/descriptions/photos/categories, real prices, availability, options and preparation estimates. Remove the sample items and sample notice.
- **Delivery:** exact service areas/postcodes, delivery fee, free-delivery threshold, delivery estimate and any order minimums. Confirm that you can serve every area listed.
- **Bank and merchant:** open a merchant account with a payment provider that supports Indian merchant UPI collection and settlement to your bank. Complete its business/KYC checks and add the bank account for settlements directly in the provider dashboard.
- **Provider:** create Razorpay API credentials in the merchant dashboard; set `RAZORPAY_KEY_ID`, `RAZORPAY_KEY_SECRET` and `RAZORPAY_WEBHOOK_SECRET` as server environment secrets. Use the provider's live keys only after a successful test-mode integration. Switch `DEMO_MODE=false` only after HTTPS, verification email, provider webhook and settlement settings are confirmed. Secret values belong only in the server environment and must never be put in browser files or the owner dashboard.
- **Customer verification:** outside demo, connect email delivery credentials for verification messages before enabling registration. The demo verification code is intentionally shown on screen; a real deployment must deliver codes through a provider and must not reveal them in logs or responses.

Razorpay checkout is started on the server with the order amount calculated from current server-side menu prices. The backend verifies the checkout HMAC and fetches payment status from Razorpay before accepting a captured payment. It also accepts signed webhooks at https://YOUR-DOMAIN/api/payments/webhook: configure that URL in the Razorpay Dashboard under Account & Settings → Webhooks, subscribe to payment.captured and payment.failed, and enter a webhook secret. Set that exact secret as RAZORPAY_WEBHOOK_SECRET in the app's server environment. The receiver validates the signature against the raw request body, matches order ID, amount and currency, and safely ignores duplicate event IDs. Webhooks require a public HTTPS URL; configure test-mode and live-mode webhooks separately in Razorpay. No refund flow is included. Do not collect or store UPI PINs or payment credentials; Razorpay handles the payment UI.

## Owner and customer tools

- Owner area: add/edit/remove items, set prices/categories/options/photos, mark items unavailable, manage order statuses, pause ordering, set hours, shop contact, fees, estimates, delivery areas and cash availability.
- Customer account: verified email signup, password login, address book, past order/payment status, reorder, current order tracking and account deletion. Guest checkout remains available. Deleting an account removes the saved profile and strips personal details from retained order records.
- Install: on mobile Chrome/Edge or desktop Chrome/Edge, use the browser's **Install app** option. HTTPS is required for install outside localhost.
- Menu photos are bundled sample food images and local SVG fallbacks, so the menu still looks complete when an external photo host is unavailable. Replace the samples with your own licensed business photos before launch. The PWA caches the app shell only; customer data and API calls are never cached.

## Notes

- Demo data is created on first launch. If you want a clean demo again, stop the server and remove `data/store.json` (this erases all demo accounts and orders).
- The app does not persist UPI PINs or payment credentials.
- This starter is not yet a substitute for a production security review, managed database backups, a formal privacy policy, email delivery configuration and a production security review.

## Change the owner password

After completing owner setup and signing in, open **Owner sign in** → **Settings** → **Change owner password**. Enter the current password and a new password of at least 14 characters. The server checks the current password and saves only a scrypt password hash. `OWNER_PASSWORD` in `.env` is only the first-time setup key; it is not the ongoing owner login password.

If you cannot sign in because you forgot the current owner password, choose **Forgot password? Reset with setup key** on the owner sign-in form. The reset requires the owner email and `OWNER_PASSWORD` setup key from `.env`, then saves a new password of at least 14 characters. Restart the server after updating app files.

For real account verification emails, configure RESEND_API_KEY and EMAIL_FROM for a verified sender in Resend. The official Resend API accepts authenticated email requests at https://api.resend.com/emails with rom, 	o, subject, and 	ext fields ([Resend email API](https://resend.com/features/email-api)). Keep DEMO_MODE=true for on-screen demo codes; switch it off only when you have configured the provider and are ready to use the live app.


## Distance-based delivery fees

Distance pricing is optional and starts disabled so the sample shop can still use its flat demo delivery fee. To turn it on:

1. Sign in to **Owner sign in** and open **Settings**.
2. Enter the shop latitude and longitude as decimal numbers. In Google Maps, right-click the shop location and copy the displayed coordinates; enter the first number as latitude and the second as longitude.
3. Set the distance base fee, fee per kilometre and maximum delivery distance, then enable **Calculate delivery fee by customer distance** and save.
4. During delivery checkout, customers enter their delivery address and tap **Use my location for delivery quote**, then allow browser location access. The browser requires HTTPS on a public site; `localhost` is allowed for local testing.

The current quote uses straight-line GPS distance from the shop, rounds kilometres up for the per-kilometre charge, adds the base fee, and applies the existing free-delivery threshold. It is not a road-route estimate, so set a delivery radius that matches the area you can actually serve. The server independently calculates the amount and rejects locations beyond the configured radius. Exact customer coordinates are used only to calculate the quote and are not saved with the order. Leave distance pricing off to keep using the flat delivery fee.
