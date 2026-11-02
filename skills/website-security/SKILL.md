---
name: website-security
description: Check and fix the security basics of a website or web app — HTTPS, security headers, forms, cookies, secrets in the frontend, admin exposure. Use when building or launching a site, when the user shares a URL to check, or before linking a site from a public profile.
---

# Website security

Most small sites are not hacked by clever attacks. They are hacked through a missing
header, a contact form that becomes a spam relay, an API key shipped in the JavaScript
bundle, or an admin panel on the default URL. All of that is checkable in minutes.

## 1. Look from the outside

```bash
URL=https://example.com
curl -sI "$URL" | grep -i -E "strict-transport-security|content-security-policy|x-content-type-options|referrer-policy|permissions-policy|x-frame-options|set-cookie|server|x-powered-by"
curl -sI "http://${URL#https://}" | head -3      # should redirect to https
```

Free external checkers worth running: securityheaders.com, Mozilla HTTP Observatory, SSL Labs (for TLS).

## 2. Headers (target)

| Header | Recommended starting value |
|---|---|
| `Strict-Transport-Security` | `max-age=31536000; includeSubDomains` (add `preload` only when sure) |
| `Content-Security-Policy` | start with `default-src 'self'; img-src 'self' data: https:; object-src 'none'; base-uri 'self'; frame-ancestors 'none'` and loosen per real need |
| `X-Content-Type-Options` | `nosniff` |
| `Referrer-Policy` | `strict-origin-when-cross-origin` |
| `Permissions-Policy` | `camera=(), microphone=(), geolocation=()` (allow only what the site uses) |
| Remove | `Server` version strings, `X-Powered-By` |

Roll out CSP with `Content-Security-Policy-Report-Only` first if the site has third-party scripts, then switch to enforcing.

Where to set them: Vercel (`vercel.json` → `headers`), Netlify (`_headers` file), Cloudflare (Rules → Transform Rules), Nginx (`add_header ... always;`), Next.js (`headers()` in `next.config.js`).

## 3. Secrets in the frontend

Anything the browser downloads is public. Search the built output and env files:

```bash
grep -rn -E "(NEXT_PUBLIC_|VITE_|REACT_APP_|PUBLIC_)" .env* src/ 2>/dev/null
grep -rn -E "sk-[A-Za-z0-9]{20,}|AKIA[0-9A-Z]{16}|AIza[0-9A-Za-z_-]{35}" dist/ build/ .next/ out/ 2>/dev/null
```

Any key used from the browser must be one the provider designs to be public (e.g. a publishable/anon key restricted by domain or row-level security). Everything else goes behind a server route.

## 4. Forms and endpoints

- Validate and size-limit every input on the server, not only in the browser.
- Rate-limit form and auth endpoints; add a bot check (Turnstile, hCaptcha, honeypot field) on public forms.
- Contact forms: never let the user control `To:`/`From:` headers; send to a fixed address.
- CSRF protection on state-changing requests if you use cookie auth.
- Escape output; never render user input as raw HTML.
- File uploads: check type and size server-side, store outside the web root or in object storage, never execute.

## 5. Cookies and auth

- Session cookies: `Secure; HttpOnly; SameSite=Lax` (or `Strict`).
- Admin panels: not on the default path if the platform allows, behind 2FA, ideally behind an IP allowlist or SSO.
- Remove default/demo accounts; change default passwords.

## 6. Hosting and domain

- 2FA on the hosting account, the domain registrar and DNS provider; these are the real keys to the site.
- Registrar lock on the domain; auto-renew on.
- No public directory listing, no `.git/` or `.env` served (`curl -s "$URL/.git/HEAD"` and `"$URL/.env"` should be 404).
- Backups exist and have been restored at least once.

## Procedure

1. Run the outside checks against the live URL (or the staging URL).
2. Compare headers to the target table; produce the exact config for the user's host/framework.
3. Search source and build output for frontend secrets.
4. Review forms, auth and admin exposure from the code.
5. Walk the hosting/domain checklist with the user.

## Output

Findings grouped by section with severity, the config snippet for their platform, and a short list of account-level tasks (2FA, registrar lock) for the user.
