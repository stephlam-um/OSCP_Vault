# OSCP Exam Report — [Your Name]

> Use OffSec's official template as the actual submission. This is a working notes scaffold to fill DURING the exam so writing the final report is just copy-paste.

## Exam info
- OSID: 
- Date: 
- Target IPs: 

---

## Box 1: [hostname / IP]

### Proof
- local.txt: 
- proof.txt: 

### Nmap
```
<paste full nmap output>
```

### Enumeration findings
- 

### Foothold — initial access as [user]

**Vulnerability:** 

**Steps:**
1. 
2. 

**Evidence:**
- Screenshot: `screenshots/box1-foothold-1.png`
- Command output:
  ```
  ```

### Privilege escalation — [user] → [root/SYSTEM]

**Vulnerability:** 

**Steps:**
1. 
2. 

**Evidence:**
- Screenshot: `screenshots/box1-privesc-1.png`

---

## Box 2: [hostname / IP]
_(repeat structure)_

---

## Box 3: [hostname / IP]
_(repeat structure)_

---

## AD Set

### Foothold box
_(structure as above)_

### Lateral move(s)
_(each pivot/credential gathering documented)_

### Domain Controller
_(structure as above)_

---

## Screenshot conventions

- Take BEFORE each major step (e.g., before running exploit, showing target state) and AFTER (showing result)
- Always include: your IP visible (ifconfig in same terminal), command run, output
- For flags: terminal showing `cat proof.txt` AND `hostname` AND `whoami` in same screenshot
- Save to `screenshots/` folder, name systematically: `box1-step-description.png`

## Pre-submission checklist
- [ ] Every flag has a screenshot showing: hostname, whoami, cat <flag>
- [ ] Every privesc has before (low-priv whoami) and after (root/SYSTEM whoami) screenshots
- [ ] All commands shown can be reproduced step-by-step
- [ ] Cross-checked enumeration evidence supports vulnerability claim
- [ ] No reference to active HTB boxes (use only PWK/PG-Practice / generic examples in writing)
- [ ] Report compiled to PDF, opens correctly
- [ ] Uploaded before deadline (do NOT wait until last 30 min)
