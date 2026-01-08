# Subdomain Takeover Recon – Step‑by‑Step Guide

This document is a **practical, reproducible workflow** for hunting **Subdomain Takeover** vulnerabilities on a single target.
It is based on real bug‑bounty methodology and reviewer expectations (HackerOne / Bugcrowd style).

> ⚠️ Always hunt **only in scope** and follow the program’s Responsible Disclosure Policy.

---

## 0. What is Subdomain Takeover (Short)

A **subdomain takeover** occurs when:

1. A subdomain points (via DNS) to a **third‑party service**
2. The resource on that service is **deleted or unclaimed**
3. An attacker can **claim that resource** and gain control of the subdomain

Key idea:

> **Dangling DNS record + claimable service = takeover**

---

## 1. Scope & Program Validation (MANDATORY)

Before any recon:

* Confirm the target is **in scope**
* Confirm **subdomain takeover is allowed**
* Check if **resource claiming is allowed** or **DNS‑only PoC** is required

Never skip this step.

---

## 2. Subdomain Enumeration

Goal: discover as many subdomains as possible.

### Commands

```bash
subfinder -d target.com -all -silent > subs.txt
amass enum -passive -d target.com >> subs.txt
assetfinder --subs-only target.com >> subs.txt

sort -u subs.txt -o subs.txt
```

### Explanation

* `subfinder` → fast passive sources
* `amass` → additional OSINT sources
* `assetfinder` → complements missing data
* `sort -u` → removes duplicates

### Output

```
subs.txt  → full list of discovered subdomains
```

---

## 3. Extract CNAME Records (CRITICAL STEP)

Subdomain takeover **mostly happens via CNAME records**, not A records.

### Command

```bash
dnsx -l subs.txt -cname -silent > cnames.txt
```

### Explanation

* Filters only subdomains that have a **CNAME record**
* A record → IP address → usually NOT takeover
* CNAME → third‑party service → possible takeover

---

## 4. Resolve CNAME Targets (DO NOT SKIP)

We must know **where the CNAME points**.

### Command

```bash
dnsx -l cnames.txt -cname -resp -silent > cnames_resolved.txt
```

### Output format

```
sub.target.com [service.thirdparty.com]
```

This file is the **core analysis file**.

---

## 5. Classify CNAME Targets

### Common SAFE / LOW‑RISK Services

Usually **not takeover‑prone**:

* `cloudflare.net`
* `akamai.net`
* `stripecdn.com`
* `fastly.net` (unless misconfigured)
* `outlook.com`, `lync.com`, `sipdir.*`

### High‑Risk / Takeover‑Prone Services

Focus on these:

* `*.amazonaws.com` (S3)
* `*.github.io`
* `*.herokuapp.com`
* `*.azurewebsites.net`
* `*.netlify.app`
* `*.readthedocs.io`
* `*.zendesk.com` (some cases)

---

## 6. Initial HTTP Validation (Non‑Intrusive)

For each **high‑risk CNAME**, check HTTP response.

### Command

```bash
curl -I https://sub.target.com
```

### How to Interpret Status Codes

| Status                 | Meaning           | Takeover?      |
| ---------------------- | ----------------- | -------------- |
| 200 OK                 | Service is live   | ❌ No           |
| 301 / 302              | Active redirect   | ❌ No           |
| 403                    | Access restricted | ❌ Usually no   |
| 404 (service‑specific) | Resource missing  | ⚠️ Investigate |
| Cloud error page       | Unbound resource  | ✅ Possible     |

---

## 7. Service‑Specific Error Signatures

### AWS S3

* `NoSuchBucket`
* `The specified bucket does not exist`

### Azure App Service

* `404 Web Site not found`
* `This app has been stopped`
* Default Azure error page

### GitHub Pages

* `There isn't a GitHub Pages site here`

### Heroku

* `No such app`
* `There's nothing here, yet`

Only **service‑specific errors** matter.

---

## 8. Manual Validation (Policy‑Aware)

Before reporting:

* Confirm the resource is **actually unclaimed**
* Check if the service **accepts the custom domain**
* Do **NOT** host content unless policy allows

### Allowed PoC Examples

* DNS record evidence
* Service error page
* Dashboard screenshot showing domain acceptance

---

## 9. False Positives (Very Common)

These are **NOT takeovers**:

* Working redirects
* Active branded pages
* Login portals
* CDN‑served content
* Internal Microsoft infrastructure

Most reports fail here.

---

## 10. Reporting Checklist (No‑Reject Standard)

Include:

1. Affected subdomain
2. DNS record showing CNAME
3. Unclaimed service error
4. Proof of claimability (policy‑safe)
5. Clear impact explanation
6. Recommended fix

### Recommended Fix

* Remove dangling DNS record
* OR reclaim and bind the resource

---

## 11. Key Takeaways

* Enumeration ≠ vulnerability
* CNAME ≠ takeover
* Error pages matter more than tools
* Manual validation decides everything
* Fewer reports, higher quality = success

---

## 12. Disclaimer

This guide is for **ethical security research only**.
Always respect scope, policy, and legal boundaries.

---

**Happy hunting 🐞**
