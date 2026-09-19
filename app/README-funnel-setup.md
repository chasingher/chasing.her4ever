# The funnel — what's here and how to wire it up

## The flow

1. **order-pagegh.html** — your sales page, unchanged in look. On submit it now
   redirects to `select-product.html` (instead of straight to WhatsApp),
   carrying the lead's name + WhatsApp number as `?name=&wa=`.
2. **select-product.html** — picks between the two designs. Has a "see a
   finished example" link per product that shows a template preview in a
   popup, with the PIN displayed above it.
3. **build-chasingher.html** / **build-memories.html** — the customer
   builder for each design. Stripped down from your `admin.html` /
   `memories-admin.html`:
   - No save, no download, no publish, no project list/sidebar.
   - The flower reveal is locked — customers can't touch it.
   - Plain-language, step-by-step instructions at the top.
   - Music/voice sections explain in plain terms that they need an actual
     mp3 file, and mention paying extra for a custom build via WhatsApp.
   - "Preview my site" opens the finished site **inline in a popup on the
     same page** — never a new tab, never a link.
   - "I'm happy with it" saves the full project *and* the finished HTML,
     then sends them to WhatsApp with a pre-filled message. A clear notice
     block above that button explains: WhatsApp is just to pay, the site
     goes live after payment, and unpaid work is deleted after 48 hours.
4. **admin.html** / **memories-admin.html** — your existing internal
   builder tools, untouched except for one addition: they now open
   `?slug=xyz` directly, so "Edit project" links from the orders page work.
5. **orders-admin.html** — new. PIN-gated (same password as your other
   Backstage tools). Lists every submission: name, WhatsApp, PIN, product,
   time left before the 48-hour deletion, a **Download HTML** button, an
   **Edit project** link (opens the real builder pre-loaded with their
   data), and buttons to mark paid/hosted or delete.

## Before any of this works — run the SQL

Open **supabase-funnel-setup.sql** and run it in the SQL editor of the
Supabase project that `admin.html` / `memories-admin.html` already use
(`zxnqoxyyzspvznehdpnj`). It creates:

- a `submissions` table (what `orders-admin.html` reads)
- a `templates` table (what the "see an example" preview reads)
- two storage buckets, `submissions` and `templates`, with public read

**Important — two Supabase projects right now:** `order-pagegh.html` uses a
*different* Supabase project (for the `leads` table) than `admin.html` /
`memories-admin.html` (for `projects`, `memory_projects`, and now
`submissions`). That's fine as-is — the lead's name/phone just ride along
as URL params — but if you'd rather have one project for everything, say
so and I'll fold `leads` into the same one.

## Adding a template to preview

`select-product.html`'s "see a finished example" reads from the
`templates` table. To add one:
1. Build a demo project as normal in `admin.html` or `memories-admin.html`.
2. Use "Download file" to get the finished HTML.
3. Upload that HTML file to the `templates` storage bucket (Supabase
   dashboard → Storage → templates).
4. Insert a row into `templates`: `product` (`chasing-her` or `memories`),
   `label`, `pin` (matching what you set on the demo project), and
   `html_url` (the public URL of the file you just uploaded).

## The WhatsApp number

Both builder files and `orders-admin.html` use the number already in your
`thank-you.html`: **2348094169489**. Change it in one place per file if
that's not right — search for `waNumber`.

## Still to decide / do

- **48-hour auto-delete**: the SQL file adds the `expires_at` column and a
  commented manual cleanup query, but Supabase won't run it on a schedule
  by itself — you'll want `pg_cron` or a small daily script for that.
- **GitHub Pages hosting after payment**: still manual — you download the
  HTML from `orders-admin.html` and push it up yourself, same as your
  current process.
- Real auth on `orders-admin.html`/the builders: right now everything uses
  the same "PIN on the client + anon key" pattern your existing tools
  already use. Fine for a small operation; worth revisiting if this grows.
