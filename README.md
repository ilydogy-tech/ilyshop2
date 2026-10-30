# ILY Shop ♡ — Fairy Maid Café

Static frontend for GitHub Pages + Supabase database/auth/private slip storage. Mobile-first, pastel pink / baby blue / lavender / cream theme.

## Important deployment note
This package is the application source, not a live deployed shop. Real orders will not work until you create a Supabase project, run `supabase/schema.sql`, set `config.js`, and add at least one admin account. GitHub Pages cannot host a private server; Supabase is the backend. Never put a Supabase `service_role` key in this repository.

## 1. Create Supabase project
1. Create a project at https://supabase.com/ and keep the project URL and publishable/anon key.
2. Open **SQL Editor → New query**, paste all of `supabase/schema.sql`, and run it. This creates and seeds the 14 starter products, including five pastel washi-tape colors, with product codes `IL001` through `IL014`.
3. In **Authentication → Users**, create the admin user with email/password.
4. Copy the admin user's UUID. In SQL Editor run:
   ```sql
   insert into public.shop_admins(user_id) values ('PASTE-ADMIN-USER-UUID-HERE');
   ```
   The account is now allowed to use `admin.html`. Do not expose admin credentials publicly.
5. In the project, check **Storage** and confirm the `payment-slips` bucket is private.

## 2. Configure the frontend
1. Copy `config.js.example` to `config.js`.
2. Set `SUPABASE_URL` and `SUPABASE_ANON_KEY` to the project URL and publishable/anon key. Keep `LINE_ID` as `@897ljiwc`.
3. `config.js` is loaded by both `index.html` and `admin.html`. Do not use a `service_role` secret key.
4. Add `config.js` to your private source repo only if you prefer; the anon key is intended for browser use and RLS is the security boundary. Never put secrets in it.

## 3. Add products and manage orders
1. Open `admin.html` after deploying and log in with the admin user.
2. Use **Add / edit product** to create the real product catalog. A product requires name, price, and category; image URL is optional. Use image URLs hosted somewhere accessible over HTTPS.
3. Review orders, open the slip with a short-lived signed URL, and update status. Customers can query status from the shop using the five-digit order number.
4. Order numbers are allocated from a PostgreSQL sequence inside `submit_order`; allocation is centralized and concurrent orders cannot receive the same number. Numbers may have gaps if an order transaction fails.

## 4. Deploy to GitHub Pages
1. Create a GitHub repository and upload `index.html`, `style.css`, `app.js`, `admin.html`, `admin.js`, `config.js`, and `supabase/schema.sql` (README optional).
2. In GitHub repository **Settings → Pages**, choose **Deploy from a branch**, select `main` and `/ (root)`, then Save.
3. Wait for Pages to publish. Open the published root for the shop; open `/admin.html` for the admin login.

## Security and operational notes
- Product prices and totals are calculated server-side by the `submit_order` database function, not trusted from browser values.
- RLS restricts product writes and order details to users listed in `shop_admins`. Public status lookup returns only order number, status, amount, and timestamp; it does not expose customer contact, name, items, or slip path.
- Slips are in a private bucket; admin views use short-lived signed URLs.
- The upload policy allows anonymous uploads only into the private slip bucket with approved file extensions. The frontend validates MIME type and size; for production, consider adding rate limits / CAPTCHA or moving uploads behind an Edge Function to reduce abuse.
- Payment display is manual bank/PromptPay or Wallet information. This is not an automatic payment gateway; the admin checks uploaded slips and updates order status.
- Test a small real order end-to-end before advertising the shop. Also review applicable privacy and consumer-protection requirements for storing customer contact details and payment slips.

## Troubleshooting
- “Shop not configured”: `config.js` is missing or has placeholder values.
- Product list empty: run `supabase/schema.sql` in Supabase SQL Editor and confirm `products` has active rows. Without Supabase configuration, the storefront shows a local preview catalog only; checkout is intentionally disabled until the database is configured.
- Admin login rejected: ensure the Auth user exists and its UUID was inserted into `public.shop_admins`.
- RLS error: re-run the SQL script or inspect Supabase **Logs → Postgres API**.
