# 🧨 Subdomain Takeover Hunting – Complete Workflow

This repository documents my **personal subdomain takeover methodology**, including tools, commands, logic, and manual verification steps.

It serves as a **repeatable checklist** for:
- Bug Bounty
- CTFs
- Learning DNS & cloud misconfigurations

---

## 🧠 What is Subdomain Takeover?

A **subdomain takeover** occurs when a subdomain points (via DNS) to a **third-party service** (AWS, GitHub Pages, Azure, etc.) that is **no longer claimed**, allowing an attacker to register that resource and gain control over the subdomain.

Root causes:
- Dangling DNS records
- Deleted cloud resources
- Improper service deprovisioning

---

## 🛠️ Tools Used

| Tool | Purpose |
|------|--------|
| subfinder | Subdomain enumeration |
| anew | Deduplication |
| dig | Low-level DNS inspection |
| nslookup | Quick DNS checks |
| subzy | Automated takeover detection |
| subjack | Signature-based detection |
| can-i-takeover.py | Manual takeover validation |
| nuclei | Automated confirmation |

---

## 🔁 Overall Workflow

subfinder
↓
anew
↓
dig / nslookup
↓
subzy / subjack
↓
can-i-takeover.py
↓
nuclei


---

## 1️⃣ Subdomain Enumeration – subfinder

### Purpose
Collect subdomains using passive sources.

### Command
```bash
subfinder -d target.com -all -silent > subs.txt
```
Notes

-all → enable all sources

-silent → clean output

Output may contain duplicates

2️⃣ Deduplication – anew
Purpose

Remove duplicate subdomains.
```bash
cat subs.txt | anew clean_subs.txt
```

Why Important

Prevents repeated scans

Reduces false positives

Improves tool stability

3️⃣ DNS Inspection – dig (CRITICAL STEP)
Purpose

Understand how DNS resolves the subdomain.

Commands
```bash
dig sub.target.com
dig sub.target.com CNAME
dig sub.target.com A
```
Takeover Indicators

CNAME → third-party service

No A record

NXDOMAIN / empty response

Example
```bash
blog.target.com. 300 IN CNAME blog-target.s3.amazonaws.com.
```
If the bucket does not exist → possible takeover

4️⃣ Quick DNS Check – nslookup
Purpose

Simple DNS resolution check.

Command
```bash
nslookup sub.target.com
```
Red Flags
```bash
*** Can't find sub.target.com: Non-existent domain
```
CNAME exists but resource is gone → suspicious

5️⃣ Automated Detection – subzy
Purpose

Scan for known takeover fingerprints.

Command
```bash
subzy run --targets clean_subs.txt
```
Notes

Fast and maintained

Always manually verify results

6️⃣ Signature Scan – subjack
Purpose

Detect takeovers using fingerprints.

Command
```bash
subjack -w clean_subs.txt -t 50 -ssl -c fingerprints.json -o subjack.txt
```
Warning

Produces false positives

Use only as supporting evidence

7️⃣ Manual Validation – can-i-takeover.py
Purpose

Check if the service is actually claimable.

Command
```bash
python3 can-i-take-over.py -u sub.target.com
```
Insight

Confirms vulnerable providers

Distinguishes protected services

8️⃣ Automated Confirmation – nuclei
Purpose

Final validation using official templates.

Command
```bash
nuclei -l clean_subs.txt -t nuclei-templates/takeovers/ -severity medium,high
```
Why Nuclei

Trusted by bug bounty programs

Consistent results

Good PoC support

🚩 Common Takeover Providers
Provider	Risk
Amazon S3	High
GitHub Pages	High
Azure Web Apps	High
Heroku	Medium
Fastly	Medium
CloudFront	Low
❌ Common Mistakes

Trusting tools blindly

Skipping dig analysis

Reporting protected services

No proof of control

Reporting 404 pages as takeovers

✅ Valid Takeover Checklist

 CNAME points to third-party service

 Resource is unclaimed

 Error confirms missing resource

 Service allows claiming

 Proof-of-Concept available

⚠️ Disclaimer

This content is for educational purposes only.
Only test domains you own or are explicitly authorized to test.

