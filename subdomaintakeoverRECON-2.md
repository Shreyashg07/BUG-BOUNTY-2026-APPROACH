🧨 Subdomain Takeover Hunting – Complete Workflow

This repository documents my personal subdomain takeover methodology, including tools, commands, logic, and manual verification steps.
It serves as a repeatable checklist for bug bounty, CTFs, and learning DNS misconfigurations.

🧠 What is Subdomain Takeover?

A subdomain takeover occurs when a subdomain points (via DNS) to a third-party service (AWS, GitHub Pages, Azure, etc.) that is no longer claimed, allowing an attacker to register that resource and gain control over the subdomain.

This is typically caused by:

Dangling DNS records

Deleted cloud resources

Improper deprovisioning

🛠️ Tools Used
Tool	Purpose
subfinder	Subdomain enumeration
anew	Deduplication
dig	Low-level DNS inspection
nslookup	Quick DNS checks
subzy	Automated takeover detection
subjack	Signature-based detection
can-i-takeover.py	Manual takeover validation
nuclei	Automated confirmation using templates
🔁 Overall Workflow
Subdomain Enumeration
        ↓
Deduplication
        ↓
DNS Analysis
        ↓
Takeover Detection
        ↓
Manual Verification
        ↓
Automation & Proof

1️⃣ Subdomain Enumeration – subfinder
Purpose

Collect as many subdomains as possible from passive sources.

Command
subfinder -d target.com -all -silent > subs.txt

Notes

-all enables all sources

-silent removes banner noise

Output may contain duplicates → use anew

2️⃣ Deduplication – anew
Purpose

Ensure clean input for scanning tools.

Command
cat subs.txt | anew clean_subs.txt

Why it matters

Prevents redundant scanning

Reduces false positives

Avoids tool crashes

3️⃣ DNS Inspection – dig (MOST IMPORTANT)
Purpose

Understand how DNS resolves the subdomain

Commands
dig sub.target.com
dig sub.target.com CNAME
dig sub.target.com A

What to Look For

CNAME pointing to third-party services

No A record

NXDOMAIN or empty responses

Example (Potential Takeover)
blog.target.com. 300 IN CNAME blog-target.s3.amazonaws.com.


If the S3 bucket does not exist, takeover may be possible.

4️⃣ Quick DNS Check – nslookup
Purpose

Simplified DNS lookup (human-friendly)

Command
nslookup sub.target.com

Red Flags
*** Can't find sub.target.com: Non-existent domain


But CNAME still exists → suspicious configuration.

5️⃣ Automated Detection – subzy
Purpose

Scan subdomains against known takeover fingerprints.

Command
subzy run --targets clean_subs.txt

Strengths

Actively maintained

Lower false positives

Fast scanning

Limitation

Always manually verify results

6️⃣ Signature Scanning – subjack
Purpose

Legacy takeover scanner using fingerprints.

Command
subjack -w clean_subs.txt -t 50 -ssl -c fingerprints.json -o subjack.txt

Notes

Produces many false positives

Use only as secondary confirmation

7️⃣ Manual Validation – can-i-takeover.py
Purpose

Confirm whether the service is actually claimable

Command
python3 can-i-take-over.py -u sub.target.com

Output Insight

Identifies vulnerable providers

Tells if takeover is theoretically possible or protected

8️⃣ Automated Confirmation – nuclei
Purpose

Final validation using official takeover templates.

Command
nuclei -l clean_subs.txt -t nuclei-templates/takeovers/ -severity medium,high

Why nuclei is important

Trusted by bug bounty programs

Standardized detection

Good PoC evidence

🚩 Common Takeover Providers
Provider	Risk Level
Amazon S3	High
GitHub Pages	High
Azure Web Apps	High
Heroku	Medium
Fastly	Medium
CloudFront	Low (harder)
❌ Common Beginner Mistakes

Trusting tool output blindly

Skipping dig analysis

Reporting protected services

Not proving control (no PoC)

Reporting false positives

✅ Valid Takeover Checklist

Before reporting:

 CNAME points to third-party service

 Resource is unclaimed / deleted

 Error message confirms missing resource

 Service allows registration

 PoC screenshots available

📌 Learning Notes

DNS understanding > tools

Manual verification > automation

Takeover ≠ just 404 page

Proof of control is mandatory

⚠️ Disclaimer

This methodology is for educational purposes only.
Only test domains you own or are explicitly authorized to test under a bug bounty or legal scope.

⭐ Final Tip

If dig doesn’t make sense → the takeover isn’t real.
