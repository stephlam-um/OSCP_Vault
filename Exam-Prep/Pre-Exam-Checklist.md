# Pre-Exam Checklist

## Week before

- [ ] Confirm exam time and timezone
- [ ] Confirm proctoring setup (webcam, mic, ID, room scan tool)
- [ ] Re-read OffSec exam guide (rules around tools, AI, restarts)
- [ ] Verify VPN connection to OffSec lab works
- [ ] Verify Kali VM is updated, snapshotted, working
- [ ] Print or save offline: report template, exam restrictions
- [ ] Light review of notes only — no new boxes

## Day before

- [ ] No new techniques. No hard boxes. Brain rest.
- [ ] Prep snacks, water, energy drinks, coffee
- [ ] Plan meals (someone bringing food / pre-cooked / delivery on speed-dial)
- [ ] Tidy desk, clean room (proctoring will scan it)
- [ ] Charger plugged in, secondary device charged in case of laptop death
- [ ] Sleep 7-8 hours. No alcohol.

## Day of — before start

- [ ] ID ready
- [ ] Phone on do-not-disturb but reachable
- [ ] Tell housemates / family: do not disturb
- [ ] Webcam, mic tested
- [ ] Two browser windows: exam control panel + exam target list
- [ ] Notes vault open in Obsidian (allowed — your own notes are fine)
- [ ] Cheatsheets open in tabs
- [ ] Empty `/exam/` directory ready with subfolders per box
- [ ] Burp Suite open, configured
- [ ] Nmap aliased / scripts ready
- [ ] Reverse shell listener template in tmux

## During — methodology

- [ ] Read entire exam objectives BEFORE starting any box. Understand point distribution.
- [ ] AD set first (worth most points, all-or-nothing-ish)
- [ ] Standalone boxes second, easiest-looking first
- [ ] Set timer per box (e.g., 2 hours per standalone). If stuck past that → move on, come back.
- [ ] Take screenshot AT EVERY MILESTONE. Don't trust yourself to remember.
- [ ] Note credentials/hashes/findings in shared `loot.md` — useful across boxes
- [ ] First break: after 4 hours. Walk, eat, away from screen 20 min.
- [ ] Sleep break around hour 10-12 if at 70+ points. Set alarm. Sleep is leverage.

## Stuck protocol

- [ ] Re-read all enumeration output. Did you miss a port? An endpoint?
- [ ] Try the obvious thing you dismissed
- [ ] Switch to a different box for 30 min, come back fresh
- [ ] Have you actually enumerated everything? Re-run nmap with different flags
- [ ] Default creds? Common usernames?
- [ ] If 3+ hours stuck on foothold → strongly consider moving on

## Last 4 hours

- [ ] Stop hacking. Lock in current points.
- [ ] Verify EVERY flag screenshot is captured correctly
- [ ] Reproduce one exploit chain end-to-end to verify it still works
- [ ] Start report skeleton in parallel with last attack attempts

## Post-exam — report (24 hours)

- [ ] Sleep first if exam ended late. Don't write tired.
- [ ] Fill OffSec template using exam-period notes
- [ ] Re-take any missing screenshots if labs still accessible
- [ ] Compile to PDF, check formatting
- [ ] Upload via OffSec portal with several hours of buffer
- [ ] Confirm upload received
- [ ] Done. Sleep.
