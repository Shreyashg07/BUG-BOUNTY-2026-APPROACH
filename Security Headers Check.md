# 🛡️ Security Headers Check — Quick Reference (curl)

A short, practical cheat sheet for checking HTTP security headers on a domain/subdomain for bug bounty.

---

## 1. Basic curl Command

```bash
curl -I -L -A "Mozilla/5.0" https://target.com
```

### What every flag means

| Flag | Long form | What it does |
|---|---|---|
| `-I` | `--head` | Send **HEAD** request — returns only headers, no body |
| `-L` | `--location` | Follow **redirects** (3xx responses) |
| `-A` | `--user-agent` | Set User-Agent (many WAFs block default `curl/x.x.x`) |
| `-s` | `--silent` | Hide progress bar and errors (clean output) |
| `-S` | `--show-error` | Show errors even with `-s` |
| `-k` | `--insecure` | Ignore SSL cert errors (self-signed, expired) |
| `-v` | `--verbose` | Show full request + response + TLS handshake |
| `-o` | `--output` | Save body to file (use `/dev/null` to discard) |
| `-D` | `--dump-header` | Dump headers to file (use `-` for stdout) |
| `-H` | `--header` | Send a custom header (e.g., `Cookie`, `Origin`) |
| `-X` | `--request` | HTTP method (`GET`, `POST`, `OPTIONS`) |
| `-e` | `--referer` | Set the Referer header |
| `-b` | `--cookie` | Send cookies |
| `-c` | `--cookie-jar` | Save received cookies to file |
| `--compressed` | — | Ask server for gzip/br (some headers only appear here) |
| `--http2` | — | Force HTTP/2 |
| `--resolve` | — | Override DNS (`--resolve sub.target.com:443:1.2.3.4`) |

---

## 2. Example Outputs

### ✅ Good (most headers present)
```bash
$ curl -sI -L -A "Mozilla/5.0" https://github.com

HTTP/2 200
strict-transport-security: max-age=31536000; includeSubdomains; preload
x-frame-options: deny
x-content-type-options: nosniff
content-security-policy: default-src 'none'; script-src 'self'...
referrer-policy: strict-origin-when-cross-origin
permissions-policy: accelerometer=(), camera=()...
```

### ❌ Bad (headers missing — report-worthy if there's real impact)
```bash
$ curl -sI -L -A "Mozilla/5.0" https://vulnerable.example.com

HTTP/1.1 200 OK
Server: Apache/2.4.29 (Ubuntu)     ← info disclosure
X-Powered-By: PHP/7.2.24            ← info disclosure
Content-Type: text/html
Set-Cookie: session=abc123; Path=/  ← missing Secure, HttpOnly, SameSite

# No HSTS, no CSP, no X-Frame-Options, no X-Content-Type-Options
```

---

## 3. One-liner Recipes

### See headers only (no body)
```bash
curl -sI https://target.com
```

### See full request + response (debug mode)
```bash
curl -v https://target.com 2>&1 | grep -E "^[<>]"
```
> `2>&1` redirects stderr (where `-v` writes) to stdout so `grep` can filter it. `^[<>]` = lines starting with `<` (response) or `>` (request).

### Follow redirects and show ALL hops
```bash
curl -sIL -A "Mozilla/5.0" https://target.com
```

### Only extract specific headers
```bash
curl -sI https://target.com | grep -iE "strict-transport|x-frame|content-security|x-content-type|referrer-policy"
```

### Check cookie flags
```bash
curl -sI https://target.com/login | grep -i set-cookie
```

### Test CORS misconfiguration (critical for bug bounty 🎯)
```bash
curl -sI -H "Origin: https://evil.com" https://api.target.com/user \
  | grep -iE "access-control-allow-origin|access-control-allow-credentials"
```
**Vulnerable if response shows:**
```
access-control-allow-origin: https://evil.com
access-control-allow-credentials: true
```

### Test if HTTP redirects to HTTPS (HSTS sanity check)
```bash
curl -sI http://target.com
# Expect: HTTP/1.1 301 Moved Permanently → Location: https://target.com/
```

### Check a specific subdomain through a custom IP
```bash
curl -sI --resolve sub.target.com:443:1.2.3.4 https://sub.target.com
```

### Send authenticated request and inspect headers
```bash
curl -sI -b "session=YOUR_COOKIE" https://target.com/dashboard
```

### Check all HTTP methods allowed (sometimes reveals PUT/DELETE)
```bash
curl -X OPTIONS -i https://target.com
# Look for: Allow: GET, POST, OPTIONS, PUT, DELETE
```

### Save full response (headers + body) for analysis
```bash
curl -sSL -D headers.txt -o body.html -A "Mozilla/5.0" https://target.com
```

---

## 4. Bulk Check Across Subdomains

```bash
# subs.txt contains one URL per line (https://sub1.target.com, etc.)
while read -r url; do
  echo "===== $url ====="
  curl -sI -L -A "Mozilla/5.0" --max-time 10 "$url" \
    | grep -iE "strict-transport|x-frame|content-security|x-content-type|server|x-powered"
  echo ""
done < subs.txt
```

**New flag:** `--max-time 10` → abort if request takes longer than 10 seconds (stops hangs).

---

## 5. Minimal Vuln → PoC Reference

| Missing Header | Quick check | Real-world PoC |
|---|---|---|
| `X-Frame-Options` / CSP `frame-ancestors` | `curl -sI URL \| grep -i frame` | `<iframe src="URL">` inside attacker page |
| `Strict-Transport-Security` | `curl -sI URL \| grep -i strict` | Show `http://URL` loads without redirect |
| `X-Content-Type-Options` | `curl -sI URL \| grep -i nosniff` | Upload `.jpg` with HTML, browser executes it |
| Weak CORS | See CORS recipe above | `fetch(URL, {credentials:"include"})` from evil origin |
| Missing `HttpOnly` cookie | `curl -sI URL \| grep -i set-cookie` | Chain with XSS → steal cookie with `document.cookie` |
| Missing `Secure` cookie | Same | Cookie leaks if any HTTP request is made |

---

## 6. Reporting Tip (critical)

> Don't report "X-Frame-Options missing" alone. Find a **sensitive action** on that page (delete account, change email, transfer money), then report it as **Clickjacking** with a working HTML PoC.
>
> Programs pay for impact, not header checklists.

---

## 7. Tools that do this at scale

```bash
shcheck.py https://target.com                              # single target
nuclei -u https://target.com -tags headers,misconfig       # template-based
testssl.sh --headers https://target.com                    # HSTS + TLS
```

Online (no install): [securityheaders.com](https://securityheaders.com) · [csp-evaluator.withgoogle.com](https://csp-evaluator.withgoogle.com)

---

*Keep this as a quick reference — pair it with the full methodology doc for report-writing and scope triage.*
