# Terminal SVG Rewrite Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Replace the current `terminal.svg` with a clean, realistic animated bash session using real GitHub profile data.

**Architecture:** Single static SVG file with SMIL animations only. No CSS keyframes (GitHub strips them). Each command types in via a clipPath wipe reveal, then output fades in. Five commands total: whoami, ls projects/, cat languages.txt, git log, final prompt.

**Tech Stack:** SVG 1.1, SMIL `<animate>`, Tokyo Night color palette, JetBrains Mono / monospace font stack.

---

## Key Constraints

- SMIL `<animate>` only — no CSS `animation:` or `@keyframes`
- No Nerd Font / Powerline glyphs (render as boxes without the font)
- No `--` inside XML comments (invalid XML)
- No emoji in `<text>` elements
- No unicode box-drawing chars for borders — use `<rect>` / `<line>`
- All text at explicit absolute x/y — never rely on tspan flow
- `fill="freeze"` on every `<animate>` that should stay after finishing

## Layout Constants

```
SVG dimensions:   760 × 480
Title bar height: 36px  (y=0..36)
Body starts:      y=36
Font:             'JetBrains Mono','Fira Code','Cascadia Code','Menlo',monospace
Font size:        13px
Char width:       7.8px  (monospace at 13px)
Line height:      20px
PAD_X:            20px   (left margin for all text)
First baseline:   y=54   (36 titlebar + 18 leading)

Prompt string:    "prashantkoirala465@github:~$ "   (29 chars)
Prompt width:     29 × 7.8 = 226.2 → round to 226px
Command x:        20 + 226 = 246px

Colors (Tokyo Night):
  bg:        #0d1117
  titlebar:  #161b22
  border:    #21262d
  dim:       #484f58
  dimmer:    #30363d
  text:      #e6edf3
  green:     #3fb950
  blue:      #58a6ff
  teal:      #39c5cf
  orange:    #e3b341
  red:       #f78166
  purple:    #bc8cff
```

## Line Map

```
Line  y-baseline  Content
----  ----------  -------
 0      54        prompt + "whoami"
 1      74        "  Prashant Koirala"
 2      94        "  Kathmandu, Nepal  |  github.com/prashantkoirala465"
 3     114        "  37 repos  |  22 followers"
 4     134        (blank)
 5     154        prompt + "ls projects/"
 6     174        "  web-development-portfolio/   AlgoLab/   pomodoro-tui/"
 7     194        "  Appointment-Management-System/   DCS-403-Assignments-PK/"
 8     214        (blank)
 9     234        prompt + "cat languages.txt"
10     254        "  JavaScript   TypeScript   C++   C#   Python   HTML/CSS"
11     274        (blank)
12     294        prompt + "git log --oneline --graph"
13     314        "  * 3a1f2b  feat: add sorting visualizer  (AlgoLab)"
14     334        "  * 9c4e7d  feat: pomodoro timer TUI"
15     354        "  * 2b8f1a  fix: responsive layout  (web-portfolio)"
16     374        "  * d5a3c0  init: Appointment Management System"
17     394        (blank)
18     414        prompt + blinking cursor (final)
```

## Animation Timeline

```
t=0.0s   cursor1 appears at x=246, y=38 (top of line 0)
t=0.3s   cmd1 "whoami" wipes in (clip1 expands), cursor1 tracks right
t=0.9s   cursor1 disappears; whoami output group fades in (lines 1-3)
t=2.0s   cursor2 appears; cmd2 "ls projects/" wipes in
t=2.7s   cursor2 disappears; ls output (lines 6-7) fades in
t=3.6s   cursor3 appears; cmd3 "cat languages.txt" wipes in
t=4.2s   cursor3 disappears; languages output (line 10) fades in
t=5.0s   cursor4 appears; cmd4 "git log --oneline --graph" wipes in
t=5.8s   cursor4 disappears; git log output (lines 13-16) fades in
t=6.8s   prompt5 group fades in; cursor5 blinks indefinitely
```

## Cursor math

Each cursor is a `<rect width="7.8" height="14" rx="1">` positioned at the
top of the current text line: `y = baseline - 13` (13px font means cap height ~13px).

- cursor1: x=246, y=41. Animates x to 246 + (6×7.8) = 246+47 = 293 during cmd1 wipe.
- cursor2: x=246, y=141. Animates x to 246 + (12×7.8) = 246+94 = 340.
- cursor3: x=246, y=221. Animates x to 246 + (17×7.8) = 246+133 = 379.
- cursor4: x=246, y=281. Animates x to 246 + (24×7.8) = 246+187 = 433.
- cursor5: x=246, y=401. Blinks (opacity 1→0→1, 1s, indefinite).

## clipPath rects

Each clipPath rect starts at the command x, covers full right width.
x = 246, y = baseline - 14, height = 18 (covers full glyph), width animates 0 → 514.

- clip1: y=40,  begin=0.3s, dur=0.6s
- clip2: y=140, begin=2.0s, dur=0.7s
- clip3: y=220, begin=3.6s, dur=0.6s
- clip4: y=280, begin=5.0s, dur=0.8s

---

## Task 1: Write the complete terminal.svg

**File:** `terminal.svg` (overwrite existing)

**Structure to implement:**

```xml
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 760 480" width="760" height="480">
  <defs>
    <style>text { font-family: 'JetBrains Mono','Fira Code','Cascadia Code','Menlo',monospace; }</style>
    <!-- 4 clipPath elements for cmd wipe reveals -->
  </defs>

  <!-- 1. Background rect -->
  <!-- 2. Title bar rect + traffic lights + title text -->
  <!-- 3. Body rect -->

  <!-- SECTION: prompt1 (always visible at t=0) -->
  <!-- prompt text at y=54 -->
  <!-- cmd1 text clipped, opacity 0 until t=0.3s -->
  <!-- cursor1 rect, opacity 0 until t=0s, tracks during wipe, hides at t=0.9s -->

  <!-- SECTION: whoami output (opacity 0, fades in at t=0.9s) -->

  <!-- SECTION: prompt2 (opacity 0, fades in at t=2.0s) -->
  <!-- cmd2 text clipped -->
  <!-- cursor2 -->

  <!-- SECTION: ls output (opacity 0, fades in at t=2.7s) -->

  <!-- SECTION: prompt3 (opacity 0, fades in at t=3.6s) -->
  <!-- cmd3 text clipped -->
  <!-- cursor3 -->

  <!-- SECTION: languages output (opacity 0, fades in at t=4.2s) -->

  <!-- SECTION: prompt4 (opacity 0, fades in at t=5.0s) -->
  <!-- cmd4 text clipped -->
  <!-- cursor4 -->

  <!-- SECTION: git log output (opacity 0, fades in at t=5.8s) -->

  <!-- SECTION: final prompt + blinking cursor (opacity 0, fades in at t=6.8s) -->
</svg>
```

**Prompt text element pattern** (reuse for all 5 prompts):
```xml
<text x="20" y="{baseline}" font-size="13">
  <tspan fill="#484f58">prashantkoirala465</tspan><tspan fill="#30363d">@</tspan><tspan fill="#484f58">github</tspan><tspan fill="#30363d">:~$ </tspan>
</text>
```
Note: keep prompt dim/muted — the commands and output should stand out.

**Output text color rules:**
- Directory names (with /): `#58a6ff`
- Plain info values: `#e6edf3`
- Labels ("Kathmandu", "37 repos"): `#484f58` for labels, `#e6edf3` for values
- git log hash: `#e3b341`, message: `#e6edf3`, scope in parens: `#39c5cf`
- git graph `*`: `#3fb950`
- language names: `#e6edf3`

**After writing, verify:**
```bash
xmllint --noout terminal.svg && echo "valid XML"
```

**Commit:**
```bash
git add terminal.svg docs/plans/
git commit -m "feat: rewrite terminal SVG with real bash session content"
git push
```
