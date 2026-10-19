# Khyxx Digitals: fixes & setup

This is your project with the security and order-saving problems fixed. **Do these steps before you deploy.**

## Do this first (about 15 minutes)

1. **Make your GitHub repo private** (Settings → General → Danger Zone → Change visibility).
   The old version published your admin PIN, payment details and order data. Treat the old PIN as compromised.
2. **Set an admin PIN.** In Vercel: Project → Settings → Environment Variables → add
   `ADMIN_PIN` = a new PIN of **8+ characters** (not `khyxx2026`). Without it, admin login is switched off on purpose.
3. **Connect a database** so orders are actually saved. Vercel can't write to files, which is why orders were being lost.
   Vercel → Storage / Marketplace → add **Upstash Redis** (free tier) → connect it to this project.
   It adds `KV_REST_API_URL` and `KV_REST_API_TOKEN` automatically (menu names can vary slightly). Then **redeploy**.
   Until a database is connected, the site tells customers "we can't save orders right now, message us on Facebook"
   instead of pretending the order went through.
4. **Add your real delivery links.** Open `data/products.json` and, for each product, paste the Canva
   *template link* (the "Use template" link) into `deliveryUrl`. That field is private: browsers never see it, and the
   tracking page only reveals it after you mark the order **Paid**. `previewUrl` is the public demo link.
   You can also paste a link for a single order in Admin → Orders → Details → *Delivery Link*.
5. **Test it:** place a test order on your live site, check it appears in `/admin`, mark it Paid, and confirm the
   tracking page shows the link.

Optional: set `NTFY_TOPIC` to a long random word, install the free **ntfy** app on your phone and subscribe to that topic.
You'll get a push notification for every new order, payment proof and inquiry.
(Anyone who knows the topic name can read the alerts, so make it hard to guess. Alerts never include emails.)

## What was fixed

**Security**
- `data/database.json`, `server.js` and other project files were downloadable by anyone. Now only the pages and
  media folders are public; everything else returns 404.
- The admin API accepted a hardcoded token that was visible in the page source. Login now returns a signed token that
  expires after 12 hours; the PIN lives in an environment variable; wrong-PIN attempts are rate-limited.
- Canva delivery links were public. They're now private to the server and unlock only after payment.
- Prices are looked up on the server (the browser can no longer set its own price).
- Order codes are random and unguessable (`KHYXX-7K2M-9QXA`) instead of 4 digits; tracking is rate-limited and
  no longer returns private notes.
- Receipt uploads: images only (JPG/PNG/WEBP, checked by content, max ~900 KB; big screenshots are shrunk in the browser
  automatically) and are no longer public files. Only you can view them in the admin.
- Customer names/links are escaped everywhere, so a malicious name can't run code in your admin panel.
- Security headers added; the old "Admin Portal" link and "Default PIN" text are gone from public pages.

**Orders & forms**
- Checkout and the contact form no longer show a fake "success" when the server fails. Customers see a clear error
  and keep their bag.
- The "coming soon" Website + RSVP item (₱999) can no longer be bought.
- New phone alerts (optional, above) and a *Delivery Link* box in the admin order window.

**Store content**
- The two guide "PDFs" were one-line text files. They are now real 2-page PDFs.
- Homepage filter labels corrected ("Invitations", "Coming soon"); fixed the broken "View websites" scroll,
  the `<spans>` typo, the 2026 banner (now "2026 & 2027") and the past sample date (now 12 · 09 · 2027).
- Images shrank from ~8 MB to under 1 MB (WebP). Facebook/Messenger link previews now show a proper image and title.
- The demo order and demo inquiry were removed from the data file.

## Things I did NOT change (your call)

- **Testimonial carousel names** (Camille & Rafael, Nina & Miguel, ...): if these are sample text, replace them. Your real
  screenshots are stronger on their own.
- **Instagram icon** links to instagram.com, not your profile. Put your profile URL in the footer of each page.
- **Social icons** are loaded from other websites (flaticon / pngimg). Consider hosting them yourself.
- **Preview links:** all 24 "WWT" themes use the same preview link, so shoppers can't see the specific theme.
  Put each theme's own Canva preview in `previewUrl` (and in `data-details-preview` in `shop.html`).
- Prices/product text live in two places (the HTML and `data/products.json`). If you change a price, change both.
  The server always charges the price in `data/products.json`.

## Running it on your computer

```
npm install
ADMIN_PIN="choose-a-pin" node server.js        # Mac/Linux
$env:ADMIN_PIN="choose-a-pin"; node server.js  # Windows PowerShell
```
Open http://localhost:3000. Local orders are saved in `data/runtime.json` (git-ignored, never committed).

## Limits worth knowing

- Rate limiting is per server instance (best effort on Vercel).
- Receipts are stored in the database, so they're capped at ~900 KB each.
- Payment is still manual (GCash/BPI + you verify). That's fine at your size; the admin flow supports it.
