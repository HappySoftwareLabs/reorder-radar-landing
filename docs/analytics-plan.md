# Analytics plan: self-hosted Umami (RR-012)

**Status: live (approved 2026-11-13).** Umami runs on Coolify as the
service `umami` and serves `https://stats.happy.software`. Every page loads
the snippet below, and each install link sends a `data-umami-event` named
`install-<utm_campaign>`.

## Why

The install links now carry UTM parameters
(`utm_source=landing&utm_medium=web&utm_campaign=<cta>`), so the Shopify
Partner dashboard can attribute installs to a landing-page CTA.
Umami would add the step before that: page views and CTA clicks on
`radar.happy.software`.
It is self-hosted, sets no cookies and collects no personal data, so the
privacy policy needs only a short addition and no cookie banner.

## Gated steps (need approval)

1. **Approve the cost and the extra service.** Umami runs as one app
   container plus a Postgres database on the existing Coolify server.
2. **Create the service in Coolify.** Add the Umami service from Coolify's
   one-click service list (it provisions Umami with Postgres).
   Set a strong `APP_SECRET`, and keep the database private (no public
   port).
3. **Give it a domain.** Pick a subdomain, for example
   `$UMAMI_HOST` (such as `stats.happy.software`; not yet decided), add the DNS
   record, and let Coolify issue the TLS certificate.
4. **Lock down the admin.** Log in with the default admin account, change
   its password right away, and create a named user.
5. **Add the website.** In Umami, add a website for
   `radar.happy.software` and copy its website ID (`$WEBSITE_ID`).
6. **Update the privacy policy.** Add a sentence to `privacy.html` saying
   the landing site counts anonymous page views with self-hosted Umami
   (no cookies, no personal data, data stays on Happy Software servers).
7. **Add the snippet** (below) in its own PR, then check the Umami
   dashboard shows a visit after the site redeploys.

## The snippet

Add this inside `<head>`, just before `</head>`, on every page:
`index.html`, `privacy.html`, `terms.html`, `faq/index.html` and
`vs/stocky/index.html`.

```html
<script
  defer
  src="https://$UMAMI_HOST/script.js"
  data-website-id="$WEBSITE_ID"
  data-domains="radar.happy.software"
></script>
```

Replace `$UMAMI_HOST` and `$WEBSITE_ID` with the real values from steps 3
and 5.
`data-domains` stops visits from local copies or preview hosts being
counted.

### Optional: CTA click events

To count clicks on each install button, add a `data-umami-event` attribute
that matches the link's `utm_campaign` value, for example:

```html
<a href="https://apps.shopify.com/reorder-radar?utm_source=landing&utm_medium=web&utm_campaign=home-hero"
   class="btn" data-umami-event="install-home-hero">Install on Shopify</a>
```

## Out of scope

- No third-party analytics (Google Analytics, tag managers, pixels).
- This plan does not deploy anything. The landing site redeploys when
  `master` changes, so merging the snippet PR is the step that turns
  analytics on.
