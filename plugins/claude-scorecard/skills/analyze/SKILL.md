---
name: claude-scorecard
description: Analyze your Claude Code session transcripts across 8 effectiveness dimensions and generate an interactive HTML scorecard. Use when the user wants to evaluate their Claude Code usage patterns or generate a skill assessment report.
allowed-tools: Bash, Write
---

# Claude Code Scorecard Generator

You will analyze the user's Claude Code session history and generate a personalized interactive HTML scorecard showing their effectiveness across 8 dimensions.

## Step 1 — Discover Sessions

Run this to find all project session files:

```bash
find ~/.claude/projects -maxdepth 2 -name "*.jsonl" | grep -v subagents | head -60
```

Then extract user messages from them:

```bash
python3 -c "
import json, glob, os
files = glob.glob(os.path.expanduser('~/.claude/projects/*/*.jsonl'))
files = [f for f in files if 'subagents' not in f]
messages = []
for f in files:
    try:
        with open(f) as fh:
            for line in fh:
                d = json.loads(line)
                if d.get('type') == 'user' and d.get('message',{}).get('role') == 'user':
                    c = d['message'].get('content','')
                    if isinstance(c, str) and len(c.strip()) > 15:
                        messages.append(c[:600])
    except: pass
print(f'Total user messages: {len(messages)}')
print(f'Sessions analyzed: {len(files)}')
for m in messages[:80]:
    print('---')
    print(m[:400])
" 2>/dev/null
```

## Step 2 — Score Each Dimension (1–10)

Read through the extracted user messages and score the user on each of these 8 dimensions. Be honest and calibrated — most users score 5–7. Reserve 9–10 for genuinely exceptional behavior.

---

### 1. 🎯 Prompt Clarity
**What to look for:**
- Do prompts include specific error codes, user IDs, expected vs actual behavior?
- Are success criteria defined upfront ("Done when…")?
- Do feature requests follow a structured format (PRD, plan, spec)?
- Are constraints stated explicitly?

**Scale anchors:**
- 1 = Vague one-liners, no context ("fix this", "make it work")
- 5 = Some context but missing exit criteria or edge cases
- 10 = Precise specs with constraints, reproduction steps, and exit criteria

---

### 2. ⚡ Iteration Speed
**What to look for:**
- Does the user test changes immediately and report back?
- Do they catch regressions within 1–2 exchanges?
- Do they use /compact or /clear before context bloats?
- Do they split failing tasks into smaller chunks quickly?

**Scale anchors:**
- 1 = Lets errors compound across many exchanges, doesn't report back
- 5 = Catches issues eventually but takes several rounds
- 10 = Tests immediately, course-corrects within one exchange

---

### 3. 🏗️ Architecture Thinking
**What to look for:**
- Does the user create or maintain CLAUDE.md with engineering preferences?
- Do they guide folder structure, separation of concerns, data flow?
- Do they question Claude's architectural proposals?
- Do they plan verification/testing before implementing?

**Scale anchors:**
- 1 = Accepts whatever Claude generates, no docs, no preferences
- 5 = Has some preferences but doesn't encode them; reacts rather than guides
- 10 = Drives architecture with documented decisions; pre-plans testing strategy

---

### 4. 🔍 Debug Strategy
**What to look for:**
- Do they share exact error messages and log output?
- Do they provide specific user IDs and reproduction steps?
- Do they isolate variables ("DB is fine, just the API response")?
- Do they ask "when was the bug introduced"?

**Scale anchors:**
- 1 = "It's broken", no logs, no reproduction steps
- 5 = Shares some context but not enough to isolate the issue
- 10 = Structured repro: Expected / Actual / Logs / Steps / Last working state

---

### 5. 🧠 Human Judgment
**What to look for:**
- Do they override risky defaults?
- Do they question Claude's assumptions and cost estimates?
- Do they revert bad approaches proactively?
- Do they make product decisions with stated rationale?

**Scale anchors:**
- 1 = Rubber-stamps every suggestion, never pushes back
- 5 = Occasionally questions things but mostly defers
- 10 = Independently validates, overrides when needed, states reasoning

---

### 6. 📐 Scope Control
**What to look for:**
- One session = one focused outcome?
- Avoids "go through the entire codebase" mega-prompts?
- Does context ever run out mid-task?
- Clear separation between investigation and implementation?

**Scale anchors:**
- 1 = Mega-prompts trying to do everything at once; context regularly runs out
- 5 = Mostly focused but sometimes combines investigation + implementation
- 10 = Atomic tasks with clear exit criteria; investigation and implementation are separate sessions

---

### 7. 🧩 Context Management
**What to look for:**
- Do they use @ references for specific files?
- Do they check /context periodically?
- Do they use /clear and /compact proactively (not reactively)?
- Do they switch models based on task complexity?
- Do they start new sessions before context expires?

**Scale anchors:**
- 1 = Contradictory instructions, stale context, never resets
- 5 = Uses /clear sometimes but mostly reactive
- 10 = Proactive resets, precise @ references, strategic model switching

---

### 8. 🛠️ Tool Leverage
**What to look for:**
- Is there a detailed CLAUDE.md with engineering preferences?
- Is plan mode used for complex features?
- Are reusable process docs created (FEATURE_PROCESS.md, VERIFICATION.md)?
- Are slash commands, statusline, hooks, or MCP servers used?

**Scale anchors:**
- 1 = No CLAUDE.md, no slash commands, pure vanilla prompting
- 5 = Has a basic CLAUDE.md but doesn't use advanced features
- 10 = Custom skills, hooks, MCP servers, CI loops, process documentation

---

## Step 3 — Compute Summary Stats

After scoring all 8 dimensions, note:
- **Average score** (sum ÷ 8, one decimal)
- **Strongest dimension** (highest score)
- **Focus area** (lowest score)
- **Session count** (from Step 1)
- **Project names** (from the ~/.claude/projects/ directory names, cleaned up)

Write one concrete **highest-leverage recommendation** for the user's single biggest gap.

Write a **one-sentence recommendation** for each dimension based on what you observed.

---

## Step 4 — Generate the HTML Scorecard

Write the following HTML to `~/claude-scorecard.html`, replacing every `__PLACEHOLDER__` with the actual computed values:

- `__SESSION_COUNT__` → number of sessions analyzed (e.g. `68`)
- `__PROJECTS__` → comma-separated project names (e.g. `orbit-ai-backend, FitnessWrapped`)
- `__DATE__` → today's date as YYYY-MM-DD
- `__AVG_SCORE__` → average score to one decimal (e.g. `7.3`)
- `__STRONGEST_EMOJI__` → emoji of strongest dimension
- `__STRONGEST_LABEL__` → label of strongest dimension
- `__STRONGEST_VAL__` → score of strongest dimension
- `__WEAKEST_LABEL__` → label of weakest dimension
- `__WEAKEST_VAL__` → score of weakest dimension
- `__LEVERAGE_TEXT__` → your highest-leverage recommendation paragraph
- `__V_CLARITY__` → score for Prompt Clarity (1–10)
- `__V_ITERATION__` → score for Iteration Speed (1–10)
- `__V_ARCHITECTURE__` → score for Architecture (1–10)
- `__V_DEBUG__` → score for Debug Strategy (1–10)
- `__V_JUDGMENT__` → score for Human Judgment (1–10)
- `__V_SCOPE__` → score for Scope Control (1–10)
- `__V_CONTEXT__` → score for Context Mgmt (1–10)
- `__V_TOOLS__` → score for Tool Leverage (1–10)
- `__REC_CLARITY__` → one-sentence recommendation for Prompt Clarity
- `__REC_ITERATION__` → one-sentence recommendation for Iteration Speed
- `__REC_ARCHITECTURE__` → one-sentence recommendation for Architecture
- `__REC_DEBUG__` → one-sentence recommendation for Debug Strategy
- `__REC_JUDGMENT__` → one-sentence recommendation for Human Judgment
- `__REC_SCOPE__` → one-sentence recommendation for Scope Control
- `__REC_CONTEXT__` → one-sentence recommendation for Context Mgmt
- `__REC_TOOLS__` → one-sentence recommendation for Tool Leverage

HTML TEMPLATE — write this exactly, only substituting the placeholders:

```html
<!DOCTYPE html>
<html lang="en" data-theme="dark">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Claude Code Scorecard</title>
  <link href="https://fonts.googleapis.com/css2?family=JetBrains+Mono:wght@300;400;500;600;700&family=Space+Grotesk:wght@400;500;600;700&display=swap" rel="stylesheet">
  <script>(function(){var t=localStorage.getItem('cc-theme')||'dark';document.documentElement.setAttribute('data-theme',t);})();</script>
  <style>
    :root{--accent:#2563EB;--accent-rgb:37,99,235;--accent-hover:#1D4ED8;--accent-on:#ffffff;--bg-primary:#FFFFFF;--bg-secondary:#F7F8FA;--bg-tertiary:#EEF0F4;--bg-elevated:#FFFFFF;--text-primary:#0D1117;--text-secondary:#4B5563;--text-tertiary:#9CA3AF;--success:#16A34A;--error:#DC2626;--border-default:#E5E7EB;--border-subtle:#F3F4F6;--shadow-sm:0 1px 2px rgba(0,0,0,.04);--shadow-md:0 4px 12px rgba(0,0,0,.08);--shadow-brand:0 4px 20px rgba(37,99,235,.25);--c-ring-hi:#E5E7EB;--c-ring-lo:#F3F4F6;--c-axis:#D1D5DB;--c-score:#0D1117;--c-dim:#9CA3AF;}
    html[data-theme="dark"]{--accent:#58A6FF;--accent-rgb:88,166,255;--accent-hover:#79C0FF;--accent-on:#0D1117;--bg-primary:#0D1117;--bg-secondary:#161B22;--bg-tertiary:#21262D;--bg-elevated:#1C2128;--text-primary:#F0F6FC;--text-secondary:#8B949E;--text-tertiary:#6E7681;--success:#3FB950;--error:#F85149;--border-default:#30363D;--border-subtle:#21262D;--shadow-sm:0 1px 2px rgba(0,0,0,.20);--shadow-md:0 4px 12px rgba(0,0,0,.35);--shadow-brand:0 4px 20px rgba(88,166,255,.20);--c-ring-hi:#1e293b;--c-ring-lo:#161B22;--c-axis:#21262D;--c-score:#F0F6FC;--c-dim:#6E7681;}
    *{margin:0;padding:0;box-sizing:border-box;}
    body{background:var(--bg-primary);color:var(--text-primary);font-family:'JetBrains Mono',monospace;min-height:100vh;transition:background-color .25s,color .25s;}
    h1,h2,h3{font-family:'Space Grotesk',sans-serif;}
    .theme-toggle{position:fixed;top:16px;right:16px;width:40px;height:40px;border-radius:50%;border:1px solid var(--border-default);background:var(--bg-elevated);cursor:pointer;font-size:1.1rem;display:flex;align-items:center;justify-content:center;z-index:1000;box-shadow:var(--shadow-sm);transition:background-color .25s,border-color .25s;}
    .theme-toggle:hover{background:var(--bg-tertiary);}
    .container{max-width:1140px;margin:0 auto;padding:40px 24px 70px;}
    .header{text-align:center;margin-bottom:40px;}
    .header h1{font-size:2rem;font-weight:700;color:var(--accent);letter-spacing:-.5px;margin-bottom:6px;}
    .tagline{font-size:.9rem;color:var(--text-secondary);font-style:italic;margin-bottom:14px;}
    .meta-row{display:flex;justify-content:center;gap:28px;flex-wrap:wrap;}
    .meta-field{display:flex;align-items:center;gap:8px;}
    .meta-label{font-size:.8rem;color:var(--text-tertiary);text-transform:uppercase;letter-spacing:1px;}
    .meta-value{font-size:.9rem;color:var(--text-secondary);}
    .grade-section{display:flex;justify-content:center;margin-bottom:32px;}
    .grade-badge{width:120px;height:120px;border-radius:50%;display:flex;flex-direction:column;align-items:center;justify-content:center;background:var(--bg-elevated);border:3px solid var(--accent);box-shadow:0 0 40px rgba(var(--accent-rgb),.18),var(--shadow-brand);transition:background-color .25s,border-color .25s,box-shadow .25s;}
    .avg-big{font-family:'Space Grotesk',sans-serif;font-size:2.8rem;font-weight:700;color:var(--accent);line-height:1;}
    .avg-label{font-size:.6rem;color:var(--text-tertiary);letter-spacing:1px;margin-top:2px;text-transform:uppercase;}
    .pills{display:flex;justify-content:center;gap:14px;flex-wrap:wrap;margin-bottom:36px;}
    .pill{background:var(--bg-elevated);border:1px solid var(--border-default);border-radius:20px;padding:8px 18px;font-size:.85rem;color:var(--text-secondary);display:flex;align-items:center;gap:8px;transition:background-color .25s,border-color .25s;}
    .pill-val{color:var(--accent);font-weight:600;}
    .leverage-box{background:var(--bg-elevated);border:1px solid var(--border-default);border-left:3px solid var(--accent);border-radius:8px;padding:18px 22px;margin-bottom:36px;max-width:750px;margin-left:auto;margin-right:auto;transition:background-color .25s,border-color .25s;}
    .lev-title{font-family:'Space Grotesk',sans-serif;font-size:.9rem;font-weight:600;color:var(--accent);margin-bottom:8px;text-transform:uppercase;letter-spacing:.5px;}
    .lev-text{font-size:.85rem;color:var(--text-secondary);line-height:1.6;}
    .main-grid{display:grid;grid-template-columns:1fr 1fr;gap:44px;align-items:start;margin-bottom:40px;}
    @media(max-width:860px){.main-grid{grid-template-columns:1fr;}}
    .chart-wrap{display:flex;justify-content:center;}
    .chart-box{width:100%;max-width:480px;aspect-ratio:1;}
    canvas{width:100%;height:100%;display:block;}
    .scores{display:flex;flex-direction:column;gap:12px;}
    .score-row{padding:12px 16px;background:var(--bg-elevated);border-radius:8px;border:1px solid var(--border-default);transition:background-color .25s,border-color .25s;}
    .score-top{display:flex;align-items:center;gap:10px;margin-bottom:6px;}
    .score-label{font-size:.85rem;color:var(--text-secondary);min-width:140px;flex-shrink:0;display:flex;align-items:center;gap:8px;}
    .score-bar-track{flex:1;height:6px;border-radius:3px;background:var(--border-default);overflow:hidden;transition:background-color .25s;}
    .score-bar-fill{height:100%;border-radius:3px;background:var(--accent);box-shadow:0 0 8px rgba(var(--accent-rgb),.35);transition:background-color .25s;}
    .score-val{font-size:1.1rem;font-weight:600;color:var(--accent);min-width:40px;text-align:right;}
    .score-anchors{display:flex;justify-content:space-between;font-size:.65rem;padding:0 2px;}
    .anchor-lo{color:var(--error);opacity:.7;}
    .anchor-hi{color:var(--success);opacity:.7;}
    .guide-section{grid-column:1/-1;}
    .guide-header{font-family:'Space Grotesk',sans-serif;font-size:1.15rem;color:var(--accent);margin-bottom:14px;padding-bottom:8px;border-bottom:1px solid var(--border-default);}
    .guide-item{border:1px solid var(--border-default);border-radius:8px;margin-bottom:8px;overflow:hidden;transition:border-color .25s;}
    .guide-item:hover{border-color:rgba(var(--accent-rgb),.4);}
    .guide-item-head{display:flex;align-items:center;justify-content:space-between;padding:14px 16px;cursor:pointer;background:var(--bg-elevated);user-select:none;transition:background-color .25s;}
    .guide-item-head:hover{background:var(--bg-tertiary);}
    .gname{font-size:.9rem;font-weight:500;color:var(--text-primary);}
    .garrow{color:var(--text-tertiary);font-size:.8rem;transition:transform .25s;}
    .guide-item-head.open .garrow{transform:rotate(180deg);}
    .guide-body{display:none;padding:16px;background:var(--bg-secondary);font-size:.85rem;line-height:1.65;color:var(--text-secondary);}
    .guide-body.open{display:block;}
    .gdesc{margin-bottom:12px;}
    .glabel{color:var(--accent);font-weight:600;font-size:.7rem;letter-spacing:1px;text-transform:uppercase;margin-bottom:6px;display:block;}
    .guide-body ul{list-style:none;padding:0;margin-bottom:12px;}
    .guide-body ul li{padding:4px 0 4px 16px;position:relative;}
    .guide-body ul li::before{content:'>';position:absolute;left:0;color:var(--accent);font-weight:700;}
    .scale-row{display:flex;justify-content:space-between;font-size:.7rem;border-top:1px solid var(--border-default);margin-top:8px;padding-top:10px;}
    .scale-lo{color:var(--error);opacity:.7;}
    .scale-hi{color:var(--success);opacity:.7;}
    .rec-box{margin-top:12px;padding:10px 14px;background:var(--bg-primary);border-radius:6px;border-left:2px solid var(--accent);transition:background-color .25s;}
    .rec-title{font-size:.7rem;color:var(--accent);font-weight:600;text-transform:uppercase;letter-spacing:.5px;margin-bottom:5px;}
    .rec-text{font-size:.82rem;color:var(--text-secondary);line-height:1.6;}
    .share-section{grid-column:1/-1;margin-top:12px;}
    .share-bar{display:flex;gap:14px;align-items:center;margin-bottom:14px;flex-wrap:wrap;}
    .btn{font-family:'JetBrains Mono',monospace;font-size:.85rem;padding:10px 22px;border-radius:6px;border:1px solid var(--accent);background:transparent;color:var(--accent);cursor:pointer;font-weight:500;transition:background-color .25s;}
    .btn:hover{background:rgba(var(--accent-rgb),.1);}
    .btn-solid{background:var(--accent);color:var(--accent-on);font-weight:600;border-color:var(--accent);}
    .btn-solid:hover{background:var(--accent-hover);border-color:var(--accent-hover);}
    .copy-ok{font-size:.85rem;color:var(--accent);opacity:0;transition:opacity .3s;}
    .copy-ok.show{opacity:1;}
    .preview-box{background:var(--bg-secondary);border:1px solid var(--border-default);border-radius:8px;padding:18px;font-size:.75rem;line-height:1.55;color:var(--text-secondary);white-space:pre;overflow-x:auto;max-height:420px;overflow-y:auto;display:none;}
    .preview-box.show{display:block;}
    .preview-box::selection{background:rgba(var(--accent-rgb),.3);}
  </style>
</head>
<body>
<button class="theme-toggle" id="themeToggle" onclick="toggleTheme()" title="Toggle light / dark mode"><span id="themeIcon"></span></button>
<div class="container">
  <div class="header">
    <h1>CLAUDE CODE SCORECARD</h1>
    <div class="tagline">__SESSION_COUNT__-Session Analysis &middot; __DATE__</div>
    <div class="meta-row">
      <div class="meta-field"><span class="meta-label">Projects</span><span class="meta-value">__PROJECTS__</span></div>
      <div class="meta-field"><span class="meta-label">Sessions</span><span class="meta-value">__SESSION_COUNT__</span></div>
    </div>
  </div>
  <div class="grade-section">
    <div class="grade-badge">
      <span class="avg-big">__AVG_SCORE__</span>
      <span class="avg-label">avg score</span>
    </div>
  </div>
  <div class="pills">
    <div class="pill">__STRONGEST_EMOJI__ Strongest <span class="pill-val">__STRONGEST_LABEL__ (__STRONGEST_VAL__)</span></div>
    <div class="pill">📐 Focus Area <span class="pill-val">__WEAKEST_LABEL__ (__WEAKEST_VAL__)</span></div>
  </div>
  <div class="leverage-box">
    <div class="lev-title">Highest-Leverage Change</div>
    <div class="lev-text">__LEVERAGE_TEXT__</div>
  </div>
  <div class="main-grid">
    <div class="chart-wrap"><div class="chart-box"><canvas id="radar"></canvas></div></div>
    <div class="scores" id="scores"></div>
    <div class="guide-section">
      <div class="guide-header">Scoring Guide</div>
      <div id="guideItems"></div>
    </div>
    <div class="share-section">
      <div class="share-bar">
        <button class="btn" onclick="togglePreview()">Preview Scorecard</button>
        <button class="btn btn-solid" onclick="copyScorecard()">Copy Scorecard</button>
        <span class="copy-ok" id="copyOk">Copied!</span>
      </div>
      <div class="preview-box" id="previewBox"></div>
    </div>
  </div>
</div>
<script>
const dims=[
  {key:'clarity',emoji:'🎯',label:'Prompt Clarity',value:__V_CLARITY__,lo:'Vibes only',hi:'Surgical precision',desc:'How specific, well-structured, and context-rich are the prompts?',indicators:['Defines success criteria upfront','Includes error codes, user IDs, expected behavior','States constraints explicitly','Provides context: what was tried, what failed','Uses structured format for features (PRD, plan, spec)'],scaleAnchors:{lo:'1 — Vague one-liners, no context',hi:'10 — Precise specs with constraints and exit criteria'},rec:'__REC_CLARITY__'},
  {key:'iteration',emoji:'⚡',label:'Iteration Speed',value:__V_ITERATION__,lo:'Ship and forget',hi:'Tight feedback loops',desc:'How quickly does the user course-correct when something goes wrong?',indicators:['Tests changes immediately after implementation','Catches regressions within 1-2 exchanges','Uses /compact or /clear before context bloats','Splits failing tasks into smaller testable chunks'],scaleAnchors:{lo:'1 — Lets errors stack up silently',hi:'10 — Catches and corrects within one exchange'},rec:'__REC_ITERATION__'},
  {key:'architecture',emoji:'🏗️',label:'Architecture',value:__V_ARCHITECTURE__,lo:'Yolo structure',hi:'Systems thinker',desc:'Does the user guide high-level design decisions?',indicators:['Creates process docs','Defines engineering preferences in CLAUDE.md','Questions architectural proposals','Plans verification strategy before implementation'],scaleAnchors:{lo:'1 — Accepts whatever Claude generates',hi:'10 — Drives architecture with documented decisions'},rec:'__REC_ARCHITECTURE__'},
  {key:'debug',emoji:'🔍',label:'Debug Strategy',value:__V_DEBUG__,lo:'"Fix it"',hi:'Root cause hunter',desc:'When things break, does the user share error logs and isolate the issue?',indicators:['Shares exact error messages and log output','Provides specific user IDs and reproduction steps','Isolates variables systematically','Asks "when was the bug introduced"'],scaleAnchors:{lo:'1 — Just says "it\'s broken"',hi:'10 — Provides full repro with isolated variables'},rec:'__REC_DEBUG__'},
  {key:'judgment',emoji:'🧠',label:'Human Judgment',value:__V_JUDGMENT__,lo:'Auto-accept everything',hi:'Critical thinker',desc:'Does the user exercise independent thinking and override bad recommendations?',indicators:['Overrides risky defaults','Questions Claude\'s assumptions','Reverts bad approaches proactively','Makes product decisions with rationale'],scaleAnchors:{lo:'1 — Rubber-stamps every suggestion',hi:'10 — Independently validates and overrides when needed'},rec:'__REC_JUDGMENT__'},
  {key:'scope',emoji:'📐',label:'Scope Control',value:__V_SCOPE__,lo:'Kitchen sink prompts',hi:'Laser-focused tasks',desc:'Does the user keep tasks focused and manageable?',indicators:['One session = one focused outcome','Avoids "go through entire codebase" mega-prompts','Context never runs out mid-task','Clear separation between investigation and implementation'],scaleAnchors:{lo:'1 — Mega-prompts that exhaust context',hi:'10 — Atomic tasks with clear exit criteria'},rec:'__REC_SCOPE__'},
  {key:'context',emoji:'🧩',label:'Context Mgmt',value:__V_CONTEXT__,lo:'Context amnesia',hi:'Surgical context control',desc:'Does the user manage Claude\'s context well?',indicators:['Uses @ references for specific files','Checks /context periodically','Uses /clear and /compact proactively','Switches models based on task complexity'],scaleAnchors:{lo:'1 — Contradictory instructions, stale context',hi:'10 — Proactive resets, precise references'},rec:'__REC_CONTEXT__'},
  {key:'tools',emoji:'🛠️',label:'Tool Leverage',value:__V_TOOLS__,lo:'Vanilla prompting',hi:'Full toolkit deployed',desc:'Does the user effectively use CLAUDE.md, slash commands, and project setup?',indicators:['Maintains detailed CLAUDE.md','Uses plan mode for complex features','Created reusable process docs','Uses statusline, model switching, hooks'],scaleAnchors:{lo:'1 — No CLAUDE.md, no slash commands',hi:'10 — Custom skills, MCP servers, CI loops'},rec:'__REC_TOOLS__'},
];
function cssVar(n){return getComputedStyle(document.documentElement).getPropertyValue(n).trim();}
function computeStats(){const v=dims.map(d=>d.value);const avg=v.reduce((a,b)=>a+b,0)/v.length;return{avg,strongest:dims[v.indexOf(Math.max(...v))],weakest:dims[v.indexOf(Math.min(...v))]};}
function render(){
  const scoresEl=document.getElementById('scores');
  scoresEl.innerHTML='';
  dims.forEach(d=>{
    const row=document.createElement('div');
    row.className='score-row';
    row.innerHTML=`<div class="score-top"><span class="score-label"><span>${d.emoji}</span>${d.label}</span><div class="score-bar-track"><div class="score-bar-fill" style="width:${d.value*10}%"></div></div><span class="score-val">${d.value}/10</span></div><div class="score-anchors"><span class="anchor-lo">${d.lo}</span><span class="anchor-hi">${d.hi}</span></div>`;
    scoresEl.appendChild(row);
  });
  const guideEl=document.getElementById('guideItems');
  guideEl.innerHTML='';
  dims.forEach(d=>{
    const item=document.createElement('div');
    item.className='guide-item';
    item.innerHTML=`<div class="guide-item-head" onclick="this.classList.toggle('open');this.nextElementSibling.classList.toggle('open');"><span class="gname">${d.emoji} ${d.label}</span><span class="garrow">▼</span></div><div class="guide-body"><div class="gdesc">${d.desc}</div><span class="glabel">What to look for</span><ul>${d.indicators.map(x=>'<li>'+x+'</li>').join('')}</ul><div class="scale-row"><span class="scale-lo">${d.scaleAnchors.lo}</span><span class="scale-hi">${d.scaleAnchors.hi}</span></div><div class="rec-box"><div class="rec-title">Recommendation</div><div class="rec-text">${d.rec}</div></div></div>`;
    guideEl.appendChild(item);
  });
  drawRadar();
}
const canvas=document.getElementById('radar');
const ctx=canvas.getContext('2d');
function drawRadar(){
  const rect=canvas.parentElement.getBoundingClientRect();
  const dpr=window.devicePixelRatio||1;
  canvas.width=rect.width*dpr;canvas.height=rect.height*dpr;
  ctx.setTransform(dpr,0,0,dpr,0,0);
  const W=rect.width,H=rect.height,cx=W/2,cy=H/2;
  const maxR=Math.min(cx,cy)*.66,n=dims.length,step=(2*Math.PI)/n,start=-Math.PI/2;
  const ac=cssVar('--accent'),acRgb=cssVar('--accent-rgb');
  const ringHi=cssVar('--c-ring-hi'),ringLo=cssVar('--c-ring-lo'),axisC=cssVar('--c-axis');
  const scoreC=cssVar('--c-score'),dimC=cssVar('--c-dim');
  ctx.clearRect(0,0,W,H);
  for(let r=1;r<=10;r++){const rad=(r/10)*maxR;ctx.beginPath();for(let i=0;i<=n;i++){const a=start+(i%n)*step;i===0?ctx.moveTo(cx+rad*Math.cos(a),cy+rad*Math.sin(a)):ctx.lineTo(cx+rad*Math.cos(a),cy+rad*Math.sin(a));}ctx.closePath();ctx.strokeStyle=r%5===0?ringHi:ringLo;ctx.lineWidth=r%5===0?1.2:.5;ctx.stroke();}
  for(let i=0;i<n;i++){const a=start+i*step;ctx.beginPath();ctx.moveTo(cx,cy);ctx.lineTo(cx+maxR*Math.cos(a),cy+maxR*Math.sin(a));ctx.strokeStyle=axisC;ctx.lineWidth=.8;ctx.stroke();}
  ctx.beginPath();for(let i=0;i<=n;i++){const idx=i%n,a=start+idx*step,r=(dims[idx].value/10)*maxR;i===0?ctx.moveTo(cx+r*Math.cos(a),cy+r*Math.sin(a)):ctx.lineTo(cx+r*Math.cos(a),cy+r*Math.sin(a));}ctx.closePath();
  const grad=ctx.createRadialGradient(cx,cy,0,cx,cy,maxR);grad.addColorStop(0,`rgba(${acRgb},.22)`);grad.addColorStop(1,`rgba(${acRgb},.04)`);ctx.fillStyle=grad;ctx.fill();
  ctx.strokeStyle=ac;ctx.lineWidth=2.5;ctx.shadowColor=`rgba(${acRgb},.5)`;ctx.shadowBlur=14;ctx.stroke();ctx.shadowBlur=0;
  for(let i=0;i<n;i++){const a=start+i*step,r=(dims[i].value/10)*maxR,x=cx+r*Math.cos(a),y=cy+r*Math.sin(a);ctx.beginPath();ctx.arc(x,y,5,0,2*Math.PI);ctx.fillStyle=ac;ctx.shadowColor=`rgba(${acRgb},.6)`;ctx.shadowBlur=10;ctx.fill();ctx.shadowBlur=0;ctx.fillStyle=scoreC;ctx.font='700 13px "JetBrains Mono",monospace';ctx.textAlign='center';ctx.textBaseline='middle';ctx.fillText(dims[i].value,x+18*Math.cos(a),y+18*Math.sin(a));ctx.font='16px sans-serif';ctx.fillText(dims[i].emoji,cx+(maxR+24)*Math.cos(a),cy+(maxR+24)*Math.sin(a));ctx.fillStyle=dimC;ctx.font='500 10px "JetBrains Mono",monospace';ctx.textAlign=Math.cos(a)<-.1?'right':Math.cos(a)>.1?'left':'center';ctx.fillText(['Clarity','Iteration','Arch','Debug','Judgment','Scope','Context','Tools'][i],cx+(maxR+44)*Math.cos(a),cy+(maxR+44)*Math.sin(a));}
}
function setThemeIcon(){document.getElementById('themeIcon').textContent=document.documentElement.getAttribute('data-theme')==='dark'?'☀️':'🌙';}
function toggleTheme(){const next=document.documentElement.getAttribute('data-theme')==='dark'?'light':'dark';document.documentElement.setAttribute('data-theme',next);localStorage.setItem('cc-theme',next);setThemeIcon();drawRadar();}
function buildAscii(){const s=computeStats();const bar=v=>'█'.repeat(Math.round(v*2))+'░'.repeat(20-Math.round(v*2));const lines=['┌'+'─'.repeat(46)+'┐','│  CLAUDE CODE SCORECARD                       │',`│  __SESSION_COUNT__ Sessions · __DATE__                    │`,'├'+'─'.repeat(46)+'┤'];dims.forEach(d=>{const name=`${d.emoji} ${d.label}`.padEnd(22).slice(0,22);lines.push(`│  ${name} ${bar(d.value)} ${String(d.value).padStart(2)}/10 │`);});lines.push('├'+'─'.repeat(46)+'┤');lines.push(`│  Average: ${s.avg.toFixed(1)}/10`.padEnd(47).slice(0,47)+'│');lines.push(`│  Strongest: ${(s.strongest.emoji+' '+s.strongest.label+' ('+s.strongest.value+')').padEnd(32).slice(0,32)} │`);lines.push(`│  Focus:     ${(s.weakest.emoji+' '+s.weakest.label+' ('+s.weakest.value+')').padEnd(32).slice(0,32)} │`);lines.push('└'+'─'.repeat(46)+'┘');return lines.join('\n');}
function togglePreview(){const box=document.getElementById('previewBox');box.classList.toggle('show');if(box.classList.contains('show'))box.textContent=buildAscii();}
function copyScorecard(){navigator.clipboard.writeText(buildAscii()).then(()=>{const ok=document.getElementById('copyOk');ok.classList.add('show');setTimeout(()=>ok.classList.remove('show'),2000);});}
window.addEventListener('resize',drawRadar);
setThemeIcon();
render();
</script>
</body>
</html>
```

## Step 5 — Open the File

After writing the HTML, run:

```bash
open ~/claude-scorecard.html
```

Then tell the user:
- The scorecard has been saved to `~/claude-scorecard.html`
- Summarize the scores in a compact table in the terminal
- State the single highest-leverage change they should make
