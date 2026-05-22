# OSCP Vault

Minimal Obsidian vault for OSCP prep. Designed to be expanded as you learn, not filled out upfront.

## Structure

- **Boxes/** — one note per box you root, using `_Templates/Box-Template.md`
- **Techniques/** — when you learn a new attack/enum method, write it here generically (not tied to one box). Link from box notes.
- **Cheatsheets/** — quick-reference one-liners you'll grep during the exam
- **Course-Notes/** — HTB Academy modules, PEN-200 notes
- **Exam-Prep/** — report template, pre-exam checklist
- **_Templates/** — Obsidian template plugin pulls from here

## Workflow

1. Start a box → copy `_Templates/Box-Template.md` to the right `Boxes/` folder, rename to box name
2. Fill in as you go. Commands, output, what worked, what didn't
3. When you learn a *new technique*, also write it up in `Techniques/` and link both ways
4. End of each box: 5-min reflection in the box note's "Lessons" section

## Plugins to install

- **Templater** (or core Templates plugin) — for the Box-Template
- **Obsidian Git** — auto-sync to GitHub
- **Dataview** (optional) — query boxes by status, OS, difficulty

## Conventions

- Tag boxes: `#linux` / `#windows` / `#ad`, `#easy` / `#medium` / `#hard`, `#rooted` / `#stuck` / `#assisted`
- "Assisted" means you used a hint or writeup. Track this — too many `#assisted` means you're not ready yet.

Test sync 
