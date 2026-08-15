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

## Live chat widget on the site (added 2026-08-15)

`index.html` now embeds the shared `alfera-chat-widget.js` at the bottom of `<body>`, with `data-tenant-id="chat365-001"`. This is the exact same widget file every Alfera/chat365 tenant uses — talking to the bubble on chat365's own site doubles as the product demo. Two things still need to happen on the backend before it actually works (can't be done from this repo/sandbox — needs your own Azure access):

### 1. Add the Tenants row

Azure Storage Browser → **Tenants** table → new entity:

- `PartitionKey`: `website`
- `RowKey`: `chat365-001`
- `tenantId`: `chat365-001`
- `businessName`: `chat365`
- `conversationMode`: `legacy_v2`
- `isActive`: `true` (Boolean)
- `leadCaptureEnabled`: `true` (Boolean)
- `websiteOrigin`: the exact origin(s) the site is served from — e.g. `https://chat365.com.au` once the custom domain is live, or the current `*.pages.dev` URL if you want the widget working before that. Comma-separate both if you want it working on both at once: `https://chat365.com.au, https://<your-project>.pages.dev`. Get this wrong (missing `https://`, trailing slash, wrong subdomain) and the widget looks broken with no visible error — see the Troubleshooting section of `Website-Channel-Onboarding-Guide.md`.
- `brandColor`: `#2563eb` (chat365's blue, matches the site)
- `logoUrl`: a public URL to the chat365 logo mark (square/round icon, not the wide wordmark — it gets cropped into a 56x56px circle on the chat bubble)
- `hubspotAccessToken`: omit for now unless you want this demo tenant's enquiries pushed to a real HubSpot portal

### 2. Deploy the new prompt file

`function-app/prompts/chat365-001.txt` already exists in this repo (chat365's own business identity, real plan list/pricing, and explicit rules against the claims already removed from the site — no self-serve signup, no "keep your number," no configurable business hours). Since `chat365-001` is a brand-new tenantId, it needs a deploy before the backend will find it:

```bash
cd "/Users/manjuladealwis/Development/Alfera/alfera-core/function-app"
func azure functionapp publish func-alfera-demo2 --javascript
```

### 3. Test

Load the live site, click the chat bubble (bottom-right), send a message, and check the Function App's Log Stream for `Website tenant resolved: chat365-001, ...` with no `Tenant not found` errors. Full troubleshooting steps (CORS errors, branding not showing, bubble not appearing) are in `alfera-core/docs/Website-Channel-Onboarding-Guide.md`.
