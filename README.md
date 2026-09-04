# A11y Snapshot

Live at **https://a11y.jdries.nl**

Paste a page's HTML source, get a real WCAG accessibility report — powered by
[axe-core](https://github.com/dequelabs/axe-core) (Deque Systems, MPL-2.0),
the same open-source engine behind most professional accessibility audits.
No signup, nothing uploaded to a server.

## Why paste-HTML instead of "enter a URL"

A URL-based scanner needs a *server* to fetch the target page — real
infrastructure, real ongoing cost, and a real SSRF surface (a visitor could
point it at an internal/private address). Paste-your-own-HTML sidesteps all
of that: everything runs in the visitor's own browser, nothing is ever
fetched server-side. The tradeoff is a slightly less convenient first step
(view-source + copy, not just typing a URL) for a meaningfully simpler,
safer, zero-server-cost tool.

## How it actually runs axe-core against pasted markup

axe-core validates its own `context` argument with an `instanceof`-style
check against constructors from **its own window**. A foreign iframe's
`contentDocument`, even when readable, fails that check — it's a different
realm, so cross-realm `instanceof` is always false. axe has to run *inside*
the frame being tested, not be handed that frame's document from the parent.

So: the pasted HTML renders into a sandboxed `<iframe srcdoc>`, axe-core's
own source is fetched once and injected as a real `<script>` into that
iframe, and the scan runs via that iframe's own `window.axe` — not the
parent page's copy.

## A deliberate sandbox tradeoff, stated plainly

That means the iframe needs `sandbox="allow-scripts allow-same-origin"` —
normally the risky combination (a sandboxed frame with both loses real
isolation from its parent) is something to avoid. It's used here anyway,
on purpose, because **there is nothing on this page for a malicious paste
to gain by escaping the sandbox**: no login, no cookies, no localStorage,
no API calls of any kind exist on this static page. The sandbox still
blocks top-level navigation and popups regardless.

## What it catches — and what it doesn't

Automated tools reliably catch roughly a third to half of real WCAG
issues: missing alt text, unlabeled form fields, insufficient color
contrast, missing document language, broken heading structure, invalid
ARIA. What they can't judge: whether alt text actually *describes* the
image, whether reading order makes sense, whether a keyboard-only user can
complete a real task. A clean scan is a real signal, not a compliance
guarantee.

## Deploy

Static files, no build step: `index.html` + a self-hosted `axe.min.js`
(fetched once at scan time, not bundled at build time, so upgrading the
axe-core version is a one-file swap).
