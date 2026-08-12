# chat365.com.au — site source

This is the starter site for **chat365** (the customer-facing brand for Alfera's product, operated under **Catalysticly Pty Ltd**, ABN 30 146 807 146). It's a single static page — `index.html`, no build step, same pattern as the existing `alfera-pilot-sites` repo.

`index.html` is a **placeholder/starter template** — copy, pricing, and design need to be replaced with real content before this goes live to customers. The DNS/hosting setup below can happen in parallel with content changes.

## Current domain status (checked 2026-08-05)

`chat365.com.au` is registered via Crazy Domains, still on Crazy Domains' default nameservers (`ns1.crazydomains.com` / `ns2.crazydomains.com`), serving only the registrar's default parking page. Nothing else is configured yet.

## Deployment steps

### 1. Push this folder to a new GitHub repo

From your own machine (not this sandbox — it can't push to GitHub):

```bash
cd chat365-site
git init
git add .
git commit -m "Initial chat365 starter site"
```

Create a new empty repo on GitHub (e.g. `chat365-site`, under whichever GitHub account/org you want to own it — could be the same `Alfera-ai` org as `alfera-pilot-sites`, or a separate one under Catalysticly if you want full separation), then:

```bash
git remote add origin <your-new-repo-url>
git push -u origin main
```

### 2. Add chat365.com.au as a Cloudflare zone

1. In the Cloudflare dashboard, **Add a site** → enter `chat365.com.au`.
2. Choose the Free plan (same as `alfera.com.au`).
3. Cloudflare will scan existing DNS records and show you **two nameservers** to set at your registrar (something like `xxx.ns.cloudflare.com` / `yyy.ns.cloudflare.com`).

### 3. Update nameservers at Crazy Domains

1. Log into Crazy Domains → **My Products** → find `chat365.com.au` → **DNS / Nameservers**.
2. Replace the current `ns1.crazydomains.com` / `ns2.crazydomains.com` with the two Cloudflare nameservers from step 2.
3. Save. This can take anywhere from a few minutes to a few hours to propagate (Cloudflare will show the zone as "Pending" until it detects the change, then "Active").

### 4. Create the Cloudflare Pages project

1. Cloudflare dashboard → **Workers & Pages** → **Create application** → **Pages** → **Connect to Git**.
2. Authorize Cloudflare's GitHub integration if you haven't already, and select the new `chat365-site` repo.
3. Build settings: **no build command, no framework** — this is a plain static site. Leave the build command blank and set the output directory to `/` (root).
4. Deploy. Cloudflare gives you a temporary `*.pages.dev` URL immediately — check the page loads there before adding the custom domain.

### 5. Add the custom domain

1. In the new Pages project → **Custom domains** → **Set up a custom domain** → enter `chat365.com.au` (and optionally `www.chat365.com.au`).
2. Since the zone is already in Cloudflare (step 2–3), the CNAME is added automatically — no manual DNS record needed.
3. Once nameservers have propagated (step 3) and the domain shows **Active**, `chat365.com.au` will serve this site directly, with automatic SSL.

## After this is live

- Replace the placeholder copy, pricing section, and branding in `index.html` with real chat365 content.
- Every future `git push` to this repo auto-deploys — no manual re-upload needed.
- If chat365 should also demo the product live (an embedded chat widget on its own homepage), that reuses the existing `alfera-chat-widget.js` from `alfera-core/website-widget/` — needs a `Tenants` row for a `chat365` tenant with its own branding fields set. Worth doing once real content/branding is settled, not before.
