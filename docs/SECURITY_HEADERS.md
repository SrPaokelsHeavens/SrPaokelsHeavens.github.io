## Security Headers Deployment Notes

This site is hosted on GitHub Pages with a custom domain.

GitHub Pages does not let us set most HTTP security headers directly from this repo.
To enforce full runtime hardening, configure these at your DNS/CDN edge (for example Cloudflare):

- `Content-Security-Policy`
- `X-Content-Type-Options: nosniff`
- `Referrer-Policy: strict-origin-when-cross-origin`
- `Permissions-Policy`
- `X-Frame-Options: DENY` (or CSP `frame-ancestors`)

Current in-repo mitigation:
- A CSP is delivered via `<meta http-equiv="Content-Security-Policy">` in `BaseLayout.astro`.

Important:
- Meta CSP has limitations compared to header CSP (for example, `frame-ancestors` is not reliably enforced from meta).
- For production-grade protection, prefer edge response headers.
