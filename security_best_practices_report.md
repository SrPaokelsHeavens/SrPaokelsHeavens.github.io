# Security Best Practices Report

## Executive Summary
This repository has a generally solid baseline for a static Astro site (no known npm vulnerabilities, no use of `eval`/`new Function`, and no `postMessage` attack surface found). The primary risks are runtime hardening gaps (missing browser security headers/CSP), trust in third-party scripts/iframes, and one DOM XSS-prone pattern using `innerHTML` with browser-provided text.

The highest-impact improvements are:
1. Add a strict CSP and baseline security headers at the edge.
2. Reduce third-party script trust surface (SRI/sandbox where possible).
3. Replace `innerHTML` in voice picker rendering with safe DOM APIs.

---

## Critical Findings
No critical findings identified.

---

## High Findings

### SBP-001: Missing runtime security headers (CSP, framing, MIME hardening)
- Severity: High
- Location: Runtime response for `https://paokelsheavens.asia/` (checked on 2026-03-08)
- Evidence:
  - Present headers include cache/server metadata, but these were absent:
    - `Content-Security-Policy`
    - `X-Frame-Options`
    - `X-Content-Type-Options`
    - `Referrer-Policy`
    - `Permissions-Policy`
  - Inline scripts are heavily used in [BaseLayout.astro](/C:/Users/Hans/Documents/UMA/SrPaokelsHeavens.github.io/src/layouts/BaseLayout.astro:183), increasing XSS blast radius without CSP.
- Impact: Any injected script payload (now or future) has fewer browser-enforced barriers; clickjacking and MIME sniffing protections are also missing.
- Fix:
  - Add security headers at hosting edge/CDN (Cloudflare/GitHub Pages front proxy):
    - `Content-Security-Policy`
    - `X-Frame-Options: DENY` (or CSP `frame-ancestors 'none'`)
    - `X-Content-Type-Options: nosniff`
    - `Referrer-Policy: strict-origin-when-cross-origin`
    - `Permissions-Policy` minimal policy
  - Because there are inline scripts, migrate toward nonce/hash CSP or move scripts to external files first.
- Mitigation:
  - Until full CSP migration, at minimum deploy `frame-ancestors`, `nosniff`, and `referrer-policy`.
- False-positive notes:
  - If headers are planned in another edge layer, this finding becomes “pending deployment validation”.

---

## Medium Findings

### SBP-002: Third-party script and iframe trust surface without integrity/sandbox controls
- Severity: Medium
- Location:
  - [src/pages/donations/index.astro:83](/C:/Users/Hans/Documents/UMA/SrPaokelsHeavens.github.io/src/pages/donations/index.astro:83)
  - [src/pages/donations/index.astro:63](/C:/Users/Hans/Documents/UMA/SrPaokelsHeavens.github.io/src/pages/donations/index.astro:63)
  - [src/components/Reader/ChapterReader.astro:249](/C:/Users/Hans/Documents/UMA/SrPaokelsHeavens.github.io/src/components/Reader/ChapterReader.astro:249)
- Evidence:
  - External PayPal SDK loaded directly.
  - Ko-fi iframe embed loaded without `sandbox` restrictions.
  - External Ko-fi widget script loaded in reader.
- Impact: Third-party compromise or malicious changes in external scripts can execute in your origin context; iframe capabilities may be broader than necessary.
- Fix:
  - Add restrictive `sandbox`/`allow` attributes to iframe as compatible.
  - Where possible, limit third-party scripts to pages that strictly need them.
  - Add CSP `script-src`/`frame-src` allowlists to trusted domains only.
- Mitigation:
  - Monitor these vendors and pin versions where feasible.
- False-positive notes:
  - Payment widgets inherently require third-party execution; risk cannot be zero, only constrained.

### SBP-003: `innerHTML` used with browser-provided voice names
- Severity: Medium
- Location: [src/components/Reader/ChapterReader.astro:1026](/C:/Users/Hans/Documents/UMA/SrPaokelsHeavens.github.io/src/components/Reader/ChapterReader.astro:1026)
- Evidence:
  - `btn.innerHTML = ... ${v.name}` where `v.name` is external/browser-provided metadata.
- Impact: Potential DOM XSS vector if malicious/unexpected HTML-like content is introduced through voice metadata.
- Fix:
  - Build button content with `createElement` + `textContent` instead of template HTML string.
- Mitigation:
  - Enforce CSP/Trusted Types progressively to reduce sink abuse.
- False-positive notes:
  - In most environments voice names are benign, but security best practice is to avoid string HTML sinks.

### SBP-004: CI dependency install uses `npm install` instead of deterministic `npm ci`
- Severity: Medium
- Location: [.github/workflows/deploy.yml:27](/C:/Users/Hans/Documents/UMA/SrPaokelsHeavens.github.io/.github/workflows/deploy.yml:27)
- Evidence:
  - Build job installs dependencies via `npm install`.
- Impact: Increased risk of dependency drift in CI and less reproducible builds.
- Fix:
  - Replace with `npm ci` in CI workflow.
- Mitigation:
  - Keep lockfile committed and protected.
- False-positive notes:
  - With lockfile present this is partially mitigated, but `npm ci` remains preferred hardening.

---

## Low Findings

### SBP-005: Storage usage is extensive but currently non-sensitive (watch for future auth data)
- Severity: Low
- Location examples:
  - [src/components/Reader/ChapterReader.astro:28](/C:/Users/Hans/Documents/UMA/SrPaokelsHeavens.github.io/src/components/Reader/ChapterReader.astro:28)
  - [src/components/ui/NotificationPrompt.astro:35](/C:/Users/Hans/Documents/UMA/SrPaokelsHeavens.github.io/src/components/ui/NotificationPrompt.astro:35)
- Evidence:
  - `localStorage` used for preferences/progress/notification flags.
- Impact: If this pattern is later reused for tokens/sessions, XSS exposure risk increases.
- Fix:
  - Keep auth/session secrets out of web storage.
  - Document approved storage keys and data classes.
- Mitigation:
  - Add lint/checklist rule preventing token/session key names in storage calls.
- False-positive notes:
  - Current usage appears non-sensitive.

---

## Positive Findings
- `npm audit` currently reports 0 vulnerabilities.
- No `eval`/`new Function` usage found.
- No `postMessage` message-channel attack surface detected.
- News modal already migrated away from unsafe `innerHTML` assignment pattern.

---

## Recommended Remediation Order
1. SBP-001 (headers/CSP at edge).
2. SBP-003 (`innerHTML` sink removal in reader).
3. SBP-002 (third-party script/iframe constraints).
4. SBP-004 (`npm ci` in workflow).
5. SBP-005 (storage policy guardrails).
