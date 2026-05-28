# Pixel Consultant Agent Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Build `pixel-consultant.html` — a single-file pixel-art simulation showing a consulting agent walking through 5 stations (Brief, Skill, MCP, Build, QA), assembling one Accenture-branded hero slide layer-by-layer, with 3 toggles (Skill / MCP / Evals) demonstrating what breaks when each layer is missing.

**Architecture:** Standalone HTML file. All rendering done via 2D Canvas API at 480×280 logical resolution, scaled 2×. Sprites drawn programmatically as colored pixel rectangles — no external image assets. State machine drives a sequence of "steps" (move agent, open callout, render slide layer, etc.) each a small async function. Three checkbox toggles mutate global state that the steps read at runtime.

**Tech Stack:** HTML5 + Canvas 2D + vanilla JavaScript. No frameworks, no build step, no dependencies. Hosted on GitHub Pages.

**Spec:** `docs/superpowers/specs/2026-05-28-pixel-consultant-agent-design.md`

---

## File structure

| File | Responsibility |
|---|---|
| `pixel-consultant.html` | Single-file app: HTML structure, CSS, all JS inlined. ~1200 lines target. |
| `README.md` | One-paragraph description + link to live URL. |
| `.gitignore` | Standard ignores (`.DS_Store`, `*.log`, `node_modules/`). |

No JS modules, no separate CSS. Single-file constraint comes from spec ("standalone, no setup, just send the link").

UI text: **English** (per user decision 2026-05-28).

---

## Task 1: Repository scaffolding

**Files:**
- Create: `C:/Users/shalkin/czech/pixel-consultant-agent/pixel-consultant.html`
- Create: `C:/Users/shalkin/czech/pixel-consultant-agent/README.md`
- Create: `C:/Users/shalkin/czech/pixel-consultant-agent/.gitignore`

The git repo and `docs/` already exist (created during spec phase). This task adds the runtime files.

- [ ] **Step 1: Create `.gitignore`**

```
.DS_Store
*.log
node_modules/
```

- [ ] **Step 2: Create `README.md`**

```markdown
# pixel-consultant-agent

Pixel-art simulation showing how a consulting AI agent uses an Accenture brand skill, MCP tool servers, and an automated QA evaluator to assemble one branded slide.

Live: https://ishalkin.github.io/pixel-consultant-agent/

Standalone HTML — open `pixel-consultant.html` in a browser, no setup needed.

Inspired by [Czechitas Pixel Agent](https://ishalkin.github.io/czechitas-pixel-agent/).
```

- [ ] **Step 3: Create `pixel-consultant.html` skeleton**

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Pixel Consultant Agent</title>
<style>
:root {
  --bg:        #1a1428;
  --panel:     #241a3b;
  --border:    #3a2960;
  --fg:        #f1f1ef;
  --fg-2:      #cfcfcf;
  --fg-3:      #818180;
  --purple:    #A100FF;
  --purple-d:  #460073;
  --purple-m:  #7500C0;
  --pink:      #FF50A0;
  --green:     #2E7D32;
  --red:       #D32F2F;
}
* { box-sizing: border-box; margin: 0; padding: 0; }
html, body {
  background: var(--bg);
  color: var(--fg);
  font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", system-ui, sans-serif;
  font-size: 15px;
  line-height: 1.5;
  min-height: 100vh;
}
.wrap { max-width: 1100px; margin: 0 auto; padding: 32px 24px 64px; }
h1 { font-size: 28px; font-weight: 700; margin-bottom: 4px; }
.sub { color: var(--fg-2); margin-bottom: 24px; }
.layout { display: grid; grid-template-columns: 220px 1fr; gap: 20px; }
.rail .panel {
  background: var(--panel);
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 14px 16px;
  margin-bottom: 14px;
}
.panel h3 {
  font-size: 12px;
  text-transform: uppercase;
  letter-spacing: 0.1em;
  color: var(--fg-3);
  margin-bottom: 10px;
}
.toggle-row { display: flex; gap: 8px; align-items: center; padding: 4px 0; font-size: 14px; }
.stage-wrap {
  background: #0e0820;
  border: 1px solid var(--border);
  border-radius: 6px;
  padding: 8px;
  display: inline-block;
}
canvas {
  display: block;
  image-rendering: pixelated;
  image-rendering: crisp-edges;
  width: 960px;
  height: 560px;
}
.controls { margin-top: 12px; display: flex; gap: 10px; }
button {
  background: var(--purple);
  color: white;
  border: none;
  border-radius: 4px;
  padding: 8px 16px;
  font-size: 14px;
  cursor: pointer;
  font-weight: 600;
}
button.ghost { background: transparent; color: var(--fg-2); border: 1px solid var(--border); }
button:hover { opacity: 0.85; }
.log {
  margin-top: 12px;
  font-family: "SF Mono", Menlo, monospace;
  font-size: 12px;
  color: var(--fg-2);
  background: #0e0820;
  border: 1px solid var(--border);
  padding: 10px 14px;
  border-radius: 4px;
  height: 120px;
  overflow-y: auto;
}
.log .line { margin: 2px 0; }
.log .line.brief  { color: #FF50A0; }
.log .line.skill  { color: #C2A3FF; }
.log .line.mcp    { color: #5bc0eb; }
.log .line.build  { color: #ffd166; }
.log .line.qa     { color: #4dd072; }
.log .line.bad    { color: #ff5252; }
</style>
</head>
<body>
<div class="wrap">
  <h1>Pixel Consultant Agent</h1>
  <div class="sub">How an AI agent uses a brand skill, MCP servers, and an automated reviewer to build one slide.</div>

  <div class="layout">
    <div class="rail">
      <div class="panel">
        <h3>Layers</h3>
        <div class="toggle-row">
          <input type="checkbox" id="t-skill" checked>
          <label for="t-skill">Skill loaded</label>
        </div>
        <div class="toggle-row">
          <input type="checkbox" id="t-mcp" checked>
          <label for="t-mcp">MCP servers</label>
        </div>
        <div class="toggle-row">
          <input type="checkbox" id="t-evals" checked>
          <label for="t-evals">QA evals</label>
        </div>
      </div>
    </div>
    <div>
      <div class="stage-wrap">
        <canvas id="stage" width="480" height="280"></canvas>
      </div>
      <div class="controls">
        <button id="btn-run">Run scenario</button>
        <button id="btn-reset" class="ghost">Reset</button>
      </div>
      <div id="log" class="log"><div class="line">[idle] click "Run scenario" to start</div></div>
    </div>
  </div>
</div>
<script>
"use strict";
// All JS will be added in subsequent tasks.
const canvas = document.getElementById("stage");
const ctx = canvas.getContext("2d");
ctx.imageSmoothingEnabled = false;
ctx.fillStyle = "#241a3b";
ctx.fillRect(0, 0, 480, 280);
ctx.fillStyle = "#A100FF";
ctx.font = "10px monospace";
ctx.fillText("scaffolding ready", 10, 20);
</script>
</body>
</html>
```

- [ ] **Step 4: Verify in browser**

Open `pixel-consultant.html` in a browser. Expected:
- Page renders with "Pixel Consultant Agent" header
- Dark purple canvas with text "scaffolding ready"
- Three toggles, two buttons (Run / Reset), log area
- No console errors

- [ ] **Step 5: Commit**

```bash
cd /c/Users/shalkin/czech/pixel-consultant-agent
git add pixel-consultant.html README.md .gitignore
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Add scaffolding: HTML, CSS, empty canvas, toggles"
```

---

## Task 2: Pixel-drawing utilities

**Files:**
- Modify: `pixel-consultant.html` — replace the placeholder script block with utilities.

These primitives are used by every later task — sprites, callouts, text, slide layers all built from `px()`, `rect()`, `text()`.

- [ ] **Step 1: Replace the script block contents (everything between `<script>` and `</script>`)**

```javascript
"use strict";

const canvas = document.getElementById("stage");
const ctx = canvas.getContext("2d");
ctx.imageSmoothingEnabled = false;

const W = 480, H = 280;

// Brand-derived palette
const PAL = {
  floor:    "#2a1f44",
  floor2:   "#221833",
  wall:     "#1a1428",
  agent:    "#A100FF",
  agentSkin:"#f4d4b8",
  agentDark:"#460073",
  furniture:"#3a2960",
  furniture2:"#241a3b",
  highlight:"#C2A3FF",
  white:    "#f1f1ef",
  grey:     "#818180",
  paper:    "#fafaf7",
  red:      "#D32F2F",
  green:    "#2E7D32",
  yellow:   "#ffd166",
  pink:     "#FF50A0",
  brand:    "#A100FF",
  brandD:   "#460073",
  brandM:   "#7500C0",
};

// Single pixel
function px(x, y, c) {
  ctx.fillStyle = c;
  ctx.fillRect(x | 0, y | 0, 1, 1);
}

// Filled rect
function rect(x, y, w, h, c) {
  ctx.fillStyle = c;
  ctx.fillRect(x | 0, y | 0, w | 0, h | 0);
}

// Outlined rect (1px stroke)
function box(x, y, w, h, fill, stroke) {
  if (fill) rect(x, y, w, h, fill);
  ctx.fillStyle = stroke;
  ctx.fillRect(x, y, w, 1);
  ctx.fillRect(x, y + h - 1, w, 1);
  ctx.fillRect(x, y, 1, h);
  ctx.fillRect(x + w - 1, y, 1, h);
}

// Tiny pixel-style text (the canvas's native text will be tiny but works)
function text(s, x, y, c, size = 6) {
  ctx.fillStyle = c;
  ctx.font = `${size}px "SF Mono", Menlo, monospace`;
  ctx.textBaseline = "top";
  ctx.fillText(s, x | 0, y | 0);
}

// Center a string in a box
function textCenter(s, cx, y, c, size = 6) {
  ctx.fillStyle = c;
  ctx.font = `${size}px "SF Mono", Menlo, monospace`;
  ctx.textBaseline = "top";
  const w = ctx.measureText(s).width;
  ctx.fillText(s, (cx - w / 2) | 0, y | 0);
}

// Sleep helper for sequencing
function sleep(ms) { return new Promise(r => setTimeout(r, ms)); }

// Clear and redraw the whole frame
function clearStage() {
  rect(0, 0, W, H, PAL.wall);
}

// Quick sanity check
clearStage();
text("primitives loaded", 8, 8, PAL.white, 8);
rect(8, 24, 60, 4, PAL.brand);
box(80, 20, 40, 12, PAL.furniture, PAL.highlight);
textCenter("HELLO", 100, 22, PAL.white, 6);
```

- [ ] **Step 2: Verify in browser**

Open `pixel-consultant.html`. Expected:
- "primitives loaded" text top-left
- Purple bar
- Outlined box with "HELLO" centered

- [ ] **Step 3: Commit**

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Add pixel drawing primitives (px, rect, box, text)"
```

---

## Task 3: Office floor + furniture

**Files:**
- Modify: `pixel-consultant.html`

Draw the static office: floor, walls, and 5 furniture pieces — one per station.

- [ ] **Step 1: Replace the sanity-check section (everything after `clearStage()` in the script) with `drawOffice()` and station coordinates**

```javascript
// 5 stations along the lower walking lane (y=200)
const STATIONS = {
  inbox: { x: 60,  y: 200, label: "INBOX" },
  skill: { x: 140, y: 200, label: "SKILL" },
  mcp:   { x: 220, y: 200, label: "MCP" },
  build: { x: 300, y: 200, label: "BUILD" },
  qa:    { x: 400, y: 200, label: "QA" },
};

function drawOffice() {
  // Wall (top 60% darker)
  rect(0, 0, W, 160, PAL.wall);
  // Floor: alternating tiles
  for (let ty = 160; ty < H; ty += 16) {
    for (let tx = 0; tx < W; tx += 24) {
      const c = ((tx / 24 + ty / 16) | 0) % 2 === 0 ? PAL.floor : PAL.floor2;
      rect(tx, ty, 24, 16, c);
    }
  }

  // Wall trim
  rect(0, 158, W, 2, PAL.brandD);

  // INBOX: mailbox/desk with envelope on top
  drawDeskWithItem(STATIONS.inbox.x, STATIONS.inbox.y, "envelope");
  // SKILL: open book on a podium
  drawDeskWithItem(STATIONS.skill.x, STATIONS.skill.y, "book");
  // MCP: server rack
  drawServerRack(STATIONS.mcp.x, STATIONS.mcp.y);
  // BUILD: workstation/easel
  drawDeskWithItem(STATIONS.build.x, STATIONS.build.y, "monitor");
  // QA: clipboard stand
  drawDeskWithItem(STATIONS.qa.x, STATIONS.qa.y, "clipboard");

  // Big wall board (slide canvas) above BUILD station
  drawWallBoard(280, 50, 160, 90);

  // Station labels (pills below each station)
  for (const s of Object.values(STATIONS)) {
    drawLabel(s.label, s.x, s.y + 18);
  }
}

function drawDeskWithItem(cx, cy, item) {
  // Desk: 30 wide, 16 tall, sitting BELOW agent feet (cy is feet)
  const dx = cx - 15, dy = cy - 28;
  rect(dx, dy + 14, 30, 6, PAL.furniture);     // desk top
  rect(dx + 2, dy + 20, 4, 8, PAL.furniture2); // left leg
  rect(dx + 24, dy + 20, 4, 8, PAL.furniture2);// right leg
  rect(dx, dy + 14, 30, 1, PAL.highlight);     // edge highlight

  // Item on top
  if (item === "envelope") {
    rect(dx + 8, dy + 8, 14, 8, PAL.paper);
    box(dx + 8, dy + 8, 14, 8, null, PAL.grey);
    // envelope flap
    px(dx + 14, dy + 11, PAL.grey);
    px(dx + 15, dy + 12, PAL.grey);
    px(dx + 16, dy + 11, PAL.grey);
  } else if (item === "book") {
    rect(dx + 6, dy + 6, 18, 10, PAL.paper);
    rect(dx + 14, dy + 6, 2, 10, PAL.grey); // spine
    // text lines
    rect(dx + 8, dy + 9, 4, 1, PAL.grey);
    rect(dx + 8, dy + 11, 5, 1, PAL.grey);
    rect(dx + 17, dy + 9, 5, 1, PAL.grey);
    rect(dx + 17, dy + 11, 4, 1, PAL.grey);
  } else if (item === "monitor") {
    rect(dx + 8, dy + 2, 14, 12, PAL.furniture2);
    rect(dx + 9, dy + 3, 12, 10, PAL.brandD);
    px(dx + 14, dy + 14, PAL.furniture2);
    px(dx + 15, dy + 14, PAL.furniture2);
  } else if (item === "clipboard") {
    rect(dx + 9, dy + 4, 12, 12, PAL.paper);
    rect(dx + 13, dy + 2, 4, 4, PAL.grey); // clip
    rect(dx + 11, dy + 8, 8, 1, PAL.grey);
    rect(dx + 11, dy + 10, 6, 1, PAL.grey);
    rect(dx + 11, dy + 12, 8, 1, PAL.grey);
  }
}

function drawServerRack(cx, cy) {
  const dx = cx - 14, dy = cy - 32;
  rect(dx, dy, 28, 28, PAL.furniture2);
  box(dx, dy, 28, 28, null, PAL.highlight);
  // 4 server units with status LEDs
  for (let i = 0; i < 4; i++) {
    const sy = dy + 2 + i * 6;
    rect(dx + 2, sy, 24, 5, PAL.furniture);
    rect(dx + 4, sy + 1, 2, 1, PAL.brand);  // led
    rect(dx + 8, sy + 2, 14, 1, PAL.grey);  // slot
  }
}

function drawWallBoard(x, y, w, h) {
  // Frame
  box(x - 2, y - 2, w + 4, h + 4, PAL.furniture, PAL.highlight);
  // Empty whiteboard (slide canvas) — paper white
  rect(x, y, w, h, PAL.paper);
}

function drawLabel(s, cx, y) {
  ctx.font = '6px "SF Mono", Menlo, monospace';
  const w = ctx.measureText(s).width + 6;
  rect((cx - w / 2) | 0, y, w, 8, PAL.furniture2);
  textCenter(s, cx, y + 1, PAL.highlight, 6);
}

clearStage();
drawOffice();
```

- [ ] **Step 2: Verify in browser**

Open the file. Expected:
- Top half: dark wall with a large white wall-board centered above BUILD station
- Bottom half: tiled purple floor
- 5 pieces of furniture along the lower lane, each with the right item on top
- Station labels (INBOX, SKILL, MCP, BUILD, QA) as pills

- [ ] **Step 3: Commit**

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Add office floor, walls, 5 station furniture, wall board"
```

---

## Task 4: Agent sprite

**Files:**
- Modify: `pixel-consultant.html`

Programmatic agent sprite — business-casual silhouette with simple walk-cycle (legs alternating).

- [ ] **Step 1: Append the agent module to the script**

```javascript
const agent = {
  x: STATIONS.inbox.x,
  y: STATIONS.inbox.y,
  facing: "down",  // "up" | "down" | "left" | "right"
  step: 0,         // walk cycle frame
  thinking: false,
};

function drawAgent() {
  const x = (agent.x | 0) - 6, y = (agent.y | 0) - 22;
  // shadow
  rect(x + 2, y + 22, 8, 1, PAL.floor2);
  // body (blazer purple)
  rect(x + 3, y + 10, 6, 8, PAL.agent);
  // shirt collar
  rect(x + 4, y + 10, 4, 1, PAL.white);
  px(x + 5, y + 11, PAL.agent);
  px(x + 6, y + 11, PAL.agent);
  // head
  rect(x + 4, y + 4, 4, 6, PAL.agentSkin);
  // hair
  rect(x + 4, y + 3, 4, 2, PAL.agentDark);
  px(x + 3, y + 4, PAL.agentDark);
  px(x + 8, y + 4, PAL.agentDark);
  // eyes
  px(x + 4, y + 7, PAL.wall);
  px(x + 7, y + 7, PAL.wall);
  // arms
  rect(x + 2, y + 11, 1, 5, PAL.agent);
  rect(x + 9, y + 11, 1, 5, PAL.agent);
  // legs (walk cycle)
  if (agent.step % 2 === 0) {
    rect(x + 4, y + 18, 2, 4, PAL.agentDark);
    rect(x + 6, y + 18, 2, 4, PAL.agentDark);
  } else {
    rect(x + 3, y + 18, 2, 4, PAL.agentDark);
    rect(x + 7, y + 18, 2, 4, PAL.agentDark);
  }
  // thinking bubble
  if (agent.thinking) {
    rect(x + 11, y - 4, 8, 6, PAL.paper);
    box(x + 11, y - 4, 8, 6, null, PAL.grey);
    px(x + 14, y + 0, PAL.grey);
    px(x + 13, y - 1, PAL.grey);
    px(x + 15, y - 2, PAL.grey);
  }
}

// Move agent toward target (px); returns Promise that resolves when reached
async function walkTo(targetX, targetY, speed = 1.5) {
  const SPS = 60;  // steps per second
  while (true) {
    const dx = targetX - agent.x;
    const dy = targetY - agent.y;
    const dist = Math.hypot(dx, dy);
    if (dist < speed) {
      agent.x = targetX;
      agent.y = targetY;
      render();
      return;
    }
    agent.x += (dx / dist) * speed;
    agent.y += (dy / dist) * speed;
    agent.facing = Math.abs(dx) > Math.abs(dy) ? (dx > 0 ? "right" : "left") : (dy > 0 ? "down" : "up");
    agent.step = (agent.step + 1) % 60;
    render();
    await sleep(1000 / SPS);
  }
}

// Master render — called every frame
function render() {
  clearStage();
  drawOffice();
  drawAgent();
  drawCallouts();   // defined later
}

// Stub callouts so render works now
function drawCallouts() { /* filled in later tasks */ }

render();
```

- [ ] **Step 2: Add a temporary walk test at end of script**

```javascript
// TEMP: test walk
(async () => {
  await sleep(500);
  await walkTo(STATIONS.skill.x, STATIONS.skill.y);
  await walkTo(STATIONS.mcp.x, STATIONS.mcp.y);
  await walkTo(STATIONS.qa.x, STATIONS.qa.y);
  await walkTo(STATIONS.inbox.x, STATIONS.inbox.y);
})();
```

- [ ] **Step 3: Verify in browser**

Open the file. Expected:
- Agent sprite (purple blazer, dark hair, skin face) appears at INBOX
- After 0.5s, walks smoothly across the floor visiting all stations
- Legs alternate during walking

- [ ] **Step 4: Remove the temporary walk test, then commit**

Delete the `// TEMP: test walk` block. Final script should end at `render();`.

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Add agent sprite with walk cycle and walkTo() helper"
```

---

## Task 5: Callouts (brief, skill checklist, MCP toolbox)

**Files:**
- Modify: `pixel-consultant.html`

Callouts are the in-world UI elements that appear above stations as the agent works. Each is a paper-colored box with text, drawn programmatically.

- [ ] **Step 1: Replace the empty `drawCallouts()` with a real implementation, plus state**

Find:
```javascript
function drawCallouts() { /* filled in later tasks */ }
```

Replace with:
```javascript
const callouts = {
  brief: null,    // { x, y, lines: [...] }
  skill: null,    // { x, y, items: [{ text, checked }] }
  mcp: null,      // { x, y, calls: [{ tool, status }] }
};

function drawCallouts() {
  if (callouts.brief)  drawBriefCallout(callouts.brief);
  if (callouts.skill)  drawSkillCallout(callouts.skill);
  if (callouts.mcp)    drawMcpCallout(callouts.mcp);
}

function drawBriefCallout(c) {
  const w = 90, h = 36;
  const x = c.x - w / 2, y = c.y - 60;
  rect(x, y, w, h, PAL.paper);
  box(x, y, w, h, null, PAL.grey);
  // Tail
  rect(x + w / 2 - 1, y + h, 2, 3, PAL.paper);
  // Header bar
  rect(x, y, w, 6, PAL.pink);
  text("CLIENT BRIEF", x + 4, y + 0, PAL.white, 5);
  // Body lines
  for (let i = 0; i < c.lines.length; i++) {
    text(c.lines[i], x + 4, y + 9 + i * 6, PAL.wall, 5);
  }
}

function drawSkillCallout(c) {
  const w = 100, h = 50;
  const x = c.x - w / 2, y = c.y - 74;
  rect(x, y, w, h, PAL.paper);
  box(x, y, w, h, null, PAL.grey);
  rect(x + w / 2 - 1, y + h, 2, 3, PAL.paper);
  rect(x, y, w, 6, PAL.brand);
  text("accenture-pptx · SKILL", x + 4, y + 0, PAL.white, 5);
  for (let i = 0; i < c.items.length; i++) {
    const item = c.items[i];
    const ty = y + 9 + i * 7;
    // Checkbox
    box(x + 4, ty, 5, 5, item.checked ? PAL.green : PAL.paper, PAL.grey);
    if (item.checked) {
      px(x + 5, ty + 2, PAL.white);
      px(x + 6, ty + 3, PAL.white);
      px(x + 7, ty + 1, PAL.white);
    }
    text(item.text, x + 12, ty - 1, PAL.wall, 5);
  }
}

function drawMcpCallout(c) {
  const w = 100, h = 36;
  const x = c.x - w / 2, y = c.y - 60;
  rect(x, y, w, h, PAL.paper);
  box(x, y, w, h, null, PAL.grey);
  rect(x + w / 2 - 1, y + h, 2, 3, PAL.paper);
  rect(x, y, w, 6, "#5bc0eb");
  text("MCP TOOLBOX", x + 4, y + 0, PAL.white, 5);
  for (let i = 0; i < c.calls.length; i++) {
    const call = c.calls[i];
    const ty = y + 9 + i * 6;
    // Status dot
    let dotC = PAL.grey;
    if (call.status === "calling") dotC = PAL.yellow;
    if (call.status === "done")    dotC = PAL.green;
    if (call.status === "error")   dotC = PAL.red;
    rect(x + 4, ty + 1, 3, 3, dotC);
    text(call.tool, x + 10, ty - 1, PAL.wall, 5);
  }
}
```

- [ ] **Step 2: Add a temporary test that shows all 3 callouts**

Add at the very end of the script:
```javascript
// TEMP: callout test
callouts.brief = {
  x: STATIONS.inbox.x, y: STATIONS.inbox.y,
  lines: ["Cloud strategy", "1 slide hero", "for CFO Monday"]
};
callouts.skill = {
  x: STATIONS.skill.x, y: STATIONS.skill.y,
  items: [
    { text: "Brand colors", checked: true },
    { text: "Typography 10pt", checked: true },
    { text: "Layout margins", checked: false },
    { text: "Slide modes", checked: false },
    { text: "Editable shapes", checked: false },
  ],
};
callouts.mcp = {
  x: STATIONS.mcp.x, y: STATIONS.mcp.y,
  calls: [
    { tool: "canva.findLogo()",     status: "done" },
    { tool: "canva.findPhoto()",    status: "calling" },
    { tool: "drawio.diagram()",     status: "idle" },
  ],
};
render();
```

- [ ] **Step 3: Verify in browser**

Expected:
- Pink-banner CLIENT BRIEF callout above INBOX with 3 lines
- Purple-banner accenture-pptx SKILL callout with 5 items, first 2 checked (green)
- Cyan-banner MCP TOOLBOX callout with 3 calls (green/yellow/grey dots)

- [ ] **Step 4: Remove the temp test, commit**

Remove the temporary test block. Keep `render()` at the end.

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Add brief, skill, and MCP callouts"
```

---

## Task 6: Slide layers on the wall board

**Files:**
- Modify: `pixel-consultant.html`

The board is the slide. During BUILD, layers stack on top of it. Render based on a `slide.layers` set.

- [ ] **Step 1: Add slide state + draw functions**

Append to the script (after the callouts section, before `render()`):

```javascript
const slide = {
  // Set of layer names that have been added
  layers: new Set(),
  // Whether QA is annotating violations
  qaAnnotations: [], // { x, y, w, h, label }
};

const SLIDE_BOX = { x: 280, y: 50, w: 160, h: 90 };

function drawSlideLayers() {
  const { x, y, w, h } = SLIDE_BOX;

  // Layer order: bg, gt, logo, title, chart, callout
  if (slide.layers.has("bg")) {
    rect(x, y, w, h, PAL.paper);
    // Subtle gradient bar at top
    rect(x, y, w, 4, PAL.brandD);
    rect(x, y + 4, w, 1, PAL.brandM);
  }
  if (slide.layers.has("gt")) {
    // » mark in top-right (Accenture brand element)
    text(">", x + w - 10, y + 6, PAL.brand, 8);
  }
  if (slide.layers.has("logo")) {
    // Pixel logo: 4 vertical bars + accent
    rect(x + 6, y + 8, 2, 6, PAL.brand);
    rect(x + 9, y + 8, 2, 6, PAL.brand);
    rect(x + 12, y + 8, 2, 6, PAL.brand);
    rect(x + 15, y + 8, 2, 6, PAL.brand);
    rect(x + 6, y + 14, 11, 1, PAL.brand);
  }
  if (slide.layers.has("title")) {
    text("Cloud strategy 2026", x + 6, y + 22, PAL.wall, 7);
    rect(x + 6, y + 32, 30, 1, PAL.brand);
  }
  if (slide.layers.has("chart")) {
    // 4 bars descending
    const bx = x + 8, by = y + 42;
    const heights = [28, 22, 16, 10];
    for (let i = 0; i < 4; i++) {
      rect(bx + i * 12, by + 28 - heights[i], 8, heights[i], PAL.brand);
    }
    // axis
    rect(bx, by + 28, 56, 1, PAL.grey);
  }
  if (slide.layers.has("callout")) {
    // Highlight box top-right
    rect(x + w - 60, y + 50, 54, 22, PAL.brand);
    text("60% cost cut", x + w - 56, y + 54, PAL.white, 6);
    text("over 18 months",  x + w - 56, y + 62, PAL.white, 6);
  }

  // QA annotations: red circles + labels
  for (const a of slide.qaAnnotations) {
    drawQACircle(a);
  }
}

function drawQACircle(a) {
  // Draw a "circle" by drawing a hollow square approximation (pixelated)
  const { x, y, w, h, label } = a;
  ctx.fillStyle = PAL.red;
  // Top + bottom
  ctx.fillRect(x + 2, y, w - 4, 1);
  ctx.fillRect(x + 2, y + h - 1, w - 4, 1);
  // Sides
  ctx.fillRect(x, y + 2, 1, h - 4);
  ctx.fillRect(x + w - 1, y + 2, 1, h - 4);
  // Diagonal corners
  ctx.fillRect(x + 1, y + 1, 1, 1);
  ctx.fillRect(x + w - 2, y + 1, 1, 1);
  ctx.fillRect(x + 1, y + h - 2, 1, 1);
  ctx.fillRect(x + w - 2, y + h - 2, 1, 1);
  // Label tail going up-right
  text(label, x + w + 2, y - 4, PAL.red, 5);
}
```

- [ ] **Step 2: Hook `drawSlideLayers()` into `render()`**

Find `function render() {` and update its body to:

```javascript
function render() {
  clearStage();
  drawOffice();
  drawSlideLayers();
  drawAgent();
  drawCallouts();
}
```

- [ ] **Step 3: Add temporary test that shows all layers**

Append at the end (after `render();`):

```javascript
// TEMP: slide test
slide.layers.add("bg");
slide.layers.add("gt");
slide.layers.add("logo");
slide.layers.add("title");
slide.layers.add("chart");
slide.layers.add("callout");
slide.qaAnnotations.push({
  x: SLIDE_BOX.x + 8, y: SLIDE_BOX.y + 40, w: 50, h: 30, label: "off-brand"
});
render();
```

- [ ] **Step 4: Verify in browser**

Expected:
- Wall board now shows: title bar (purple), `>` mark, logo, "Cloud strategy 2026" title, 4 purple bars, purple callout box ("60% cost cut")
- A red rectangular outline around the chart area with "off-brand" label

- [ ] **Step 5: Remove temp test, commit**

Remove the temporary block. The script should end at `render();`.

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Add slide layer rendering on wall board with QA annotation support"
```

---

## Task 7: Scenario state machine

**Files:**
- Modify: `pixel-consultant.html`

Wire the agent walk + callouts + slide layers into a scripted scenario triggered by the Run button. This is the heart of the simulation.

- [ ] **Step 1: Add toggles state + log helpers**

Append to the script before `render();`:

```javascript
const toggles = {
  skill: true,
  mcp: true,
  evals: true,
};

document.getElementById("t-skill").addEventListener("change", e => toggles.skill = e.target.checked);
document.getElementById("t-mcp").addEventListener("change", e => toggles.mcp = e.target.checked);
document.getElementById("t-evals").addEventListener("change", e => toggles.evals = e.target.checked);

const logEl = document.getElementById("log");
function log(msg, kind = "") {
  const div = document.createElement("div");
  div.className = "line" + (kind ? " " + kind : "");
  div.textContent = msg;
  logEl.appendChild(div);
  logEl.scrollTop = logEl.scrollHeight;
}
function clearLog() { logEl.innerHTML = ""; }
```

- [ ] **Step 2: Add reset + scenario runner**

Append:
```javascript
let running = false;

function reset() {
  agent.x = STATIONS.inbox.x;
  agent.y = STATIONS.inbox.y;
  agent.thinking = false;
  callouts.brief = null;
  callouts.skill = null;
  callouts.mcp = null;
  slide.layers.clear();
  slide.qaAnnotations.length = 0;
  clearLog();
  log("[idle] click \"Run scenario\" to start");
  render();
}

async function runScenario() {
  if (running) return;
  running = true;
  reset();
  clearLog();

  // ── 1. INBOX ──
  log("walking to INBOX…");
  await walkTo(STATIONS.inbox.x, STATIONS.inbox.y);
  log("> Client e-mail: Cloud strategy, 1 slide", "brief");
  callouts.brief = {
    x: STATIONS.inbox.x, y: STATIONS.inbox.y,
    lines: ["Cloud strategy", "1 hero slide", "for CFO Monday"],
  };
  render();
  await sleep(1500);

  // ── 2. SKILL ──
  callouts.brief = null;
  log("walking to SKILL…");
  await walkTo(STATIONS.skill.x, STATIONS.skill.y);

  if (toggles.skill) {
    log("> reading accenture-pptx/SKILL.md", "skill");
    callouts.skill = {
      x: STATIONS.skill.x, y: STATIONS.skill.y,
      items: [
        { text: "Brand colors (purple A100FF)",  checked: false },
        { text: "Typography (Graphik, 10pt fl.)",checked: false },
        { text: "Layout margins (0.45 / 0.35)",  checked: false },
        { text: "Slide modes (Pres / Detail)",   checked: false },
        { text: "Editable: 1 unit = 1 shape",    checked: false },
      ],
    };
    render();
    for (let i = 0; i < callouts.skill.items.length; i++) {
      await sleep(400);
      callouts.skill.items[i].checked = true;
      log("  ✓ " + callouts.skill.items[i].text, "skill");
      render();
    }
    await sleep(800);
  } else {
    log("> SKILL OFF — agent has no brand rules", "bad");
    agent.thinking = true;
    render();
    await sleep(1500);
    agent.thinking = false;
  }
  callouts.skill = null;

  // ── 3. MCP ──
  log("walking to MCP…");
  await walkTo(STATIONS.mcp.x, STATIONS.mcp.y);
  if (toggles.mcp) {
    log("> calling MCP servers", "mcp");
    callouts.mcp = {
      x: STATIONS.mcp.x, y: STATIONS.mcp.y,
      calls: [
        { tool: "canva.findLogo()",      status: "idle" },
        { tool: "canva.findPhoto()",     status: "idle" },
        { tool: "drawio.diagram()",      status: "idle" },
      ],
    };
    render();
    for (const call of callouts.mcp.calls) {
      call.status = "calling"; render();
      log("  → " + call.tool, "mcp");
      await sleep(700);
      call.status = "done"; render();
      await sleep(200);
    }
    await sleep(600);
  } else {
    log("> MCP OFF — no asset access", "bad");
    await sleep(1500);
  }
  callouts.mcp = null;

  // ── 4. BUILD ──
  log("walking to BUILD…");
  await walkTo(STATIONS.build.x, STATIONS.build.y);
  log("> assembling slide layer by layer", "build");
  // Layer order. Each layer becomes "broken" when toggles missing.
  const layerOrder = ["bg", "gt", "logo", "title", "chart", "callout"];
  for (const layer of layerOrder) {
    await sleep(350);
    slide.layers.add(layer);
    log("  + " + layer, "build");
    render();
  }
  await sleep(600);

  // ── 5. QA ──
  log("walking to QA…");
  await walkTo(STATIONS.qa.x, STATIONS.qa.y);
  if (toggles.evals) {
    log("> accenture-pptx-qa critic running", "qa");
    await sleep(800);
    // If skill or mcp off, flag violations
    const violations = [];
    if (!toggles.skill) {
      violations.push({ x: SLIDE_BOX.x + 4, y: SLIDE_BOX.y + 18, w: 100, h: 18, label: "Calibri off-brand" });
      violations.push({ x: SLIDE_BOX.x + 4, y: SLIDE_BOX.y + 40, w: 70, h: 30, label: "8pt < 10pt floor" });
    }
    if (!toggles.mcp) {
      violations.push({ x: SLIDE_BOX.x + 4, y: SLIDE_BOX.y + 6, w: 18, h: 12, label: "missing logo" });
    }
    if (violations.length === 0) {
      log("  ✓ all 5 rule groups pass", "qa");
      // Green pass mark
      text("PASS", SLIDE_BOX.x + SLIDE_BOX.w - 28, SLIDE_BOX.y + SLIDE_BOX.h - 12, PAL.green, 8);
    } else {
      for (const v of violations) {
        slide.qaAnnotations.push(v);
        log("  ✗ " + v.label, "bad");
        render();
        await sleep(500);
      }
      log("[QA] " + violations.length + " violations — needs rework", "bad");
    }
    render();
  } else {
    log("> EVALS OFF — broken slide ships", "bad");
    await sleep(800);
    log("[client] :( this is off-brand, please redo", "bad");
  }

  log("scenario complete", "");
  running = false;
}

document.getElementById("btn-run").addEventListener("click", runScenario);
document.getElementById("btn-reset").addEventListener("click", reset);
```

- [ ] **Step 3: Verify in browser**

1. Open the file. Hit Run scenario with all 3 toggles ON. Expected:
   - Agent walks INBOX → SKILL → MCP → BUILD → QA
   - Brief callout appears, then skill checklist ticks one by one, then MCP toolbox calls light up green
   - Slide assembles layer by layer on the wall board
   - QA stamps "PASS" in green, log shows ✓ for all 5 rule groups
2. Hit Reset, turn OFF Skill, hit Run. Expected:
   - Skill stage shows "agent has no brand rules" with thinking bubble
   - QA flags violations (red rectangles) on the slide with "Calibri off-brand" and "8pt < 10pt floor"
3. Hit Reset, turn OFF Evals (rest ON), hit Run. Expected:
   - QA stage logs "EVALS OFF — broken slide ships" + "[client] :( …"
4. Hit Reset, turn OFF MCP, hit Run. Expected:
   - MCP stage logs "no asset access"
   - QA flags missing logo

- [ ] **Step 4: Commit**

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Add scenario state machine: agent walks 5 stations and assembles slide"
```

---

## Task 8: Toggle-driven broken-slide rendering

**Files:**
- Modify: `pixel-consultant.html`

When toggles are OFF, the layers should *visually* look broken (not just trigger QA flags). This makes the "what does the layer add" lesson land.

- [ ] **Step 1: Update `drawSlideLayers()` to read toggles**

Find the `function drawSlideLayers()` definition and replace the body with:

```javascript
function drawSlideLayers() {
  const { x, y, w, h } = SLIDE_BOX;
  const skillOn = toggles.skill;
  const mcpOn = toggles.mcp;

  if (slide.layers.has("bg")) {
    rect(x, y, w, h, PAL.paper);
    if (skillOn) {
      rect(x, y, w, 4, PAL.brandD);
      rect(x, y + 4, w, 1, PAL.brandM);
    } else {
      // Off-brand blue
      rect(x, y, w, 4, "#224BFF");
    }
  }
  if (slide.layers.has("gt") && skillOn) {
    text(">", x + w - 10, y + 6, PAL.brand, 8);
  }
  if (slide.layers.has("logo")) {
    if (mcpOn) {
      rect(x + 6, y + 8, 2, 6, PAL.brand);
      rect(x + 9, y + 8, 2, 6, PAL.brand);
      rect(x + 12, y + 8, 2, 6, PAL.brand);
      rect(x + 15, y + 8, 2, 6, PAL.brand);
      rect(x + 6, y + 14, 11, 1, PAL.brand);
    } else {
      // Missing asset placeholder
      box(x + 4, y + 6, 16, 10, "#eee", PAL.grey);
      text("?", x + 10, y + 7, PAL.grey, 8);
    }
  }
  if (slide.layers.has("title")) {
    if (skillOn) {
      text("Cloud strategy 2026", x + 6, y + 22, PAL.wall, 7);
      rect(x + 6, y + 32, 30, 1, PAL.brand);
    } else {
      // ALL CAPS, smaller, off-brand color
      text("CLOUD STRATEGY 2026", x + 6, y + 22, "#224BFF", 5);
    }
  }
  if (slide.layers.has("chart")) {
    const bx = x + 8, by = y + 42;
    const heights = [28, 22, 16, 10];
    const barC = skillOn ? PAL.brand : "#224BFF";
    for (let i = 0; i < 4; i++) {
      rect(bx + i * 12, by + 28 - heights[i], 8, heights[i], barC);
    }
    rect(bx, by + 28, 56, 1, PAL.grey);
  }
  if (slide.layers.has("callout")) {
    if (skillOn) {
      rect(x + w - 60, y + 50, 54, 22, PAL.brand);
      text("60% cost cut", x + w - 56, y + 54, PAL.white, 6);
      text("over 18 months",  x + w - 56, y + 62, PAL.white, 6);
    } else {
      // No fill, plain black text — looks generic
      box(x + w - 60, y + 50, 54, 22, null, PAL.grey);
      text("60% cost cut",   x + w - 56, y + 54, PAL.wall, 5);
      text("over 18 months", x + w - 56, y + 62, PAL.wall, 5);
    }
  }

  for (const a of slide.qaAnnotations) {
    drawQACircle(a);
  }
}
```

- [ ] **Step 2: Verify in browser**

1. Toggle Skill OFF, hit Run. After BUILD, the slide should look generic — blue accents, ALL-CAPS title in tiny font, plain callout outline (no purple fill).
2. Toggle Skill ON, MCP OFF, hit Run. Logo area should show grey "?" placeholder.
3. All toggles ON: full Accenture-styled slide.

- [ ] **Step 3: Commit**

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Render slide differently when Skill/MCP toggles are off"
```

---

## Task 9: Polish — header, footer, copy, button states

**Files:**
- Modify: `pixel-consultant.html`

- [ ] **Step 1: Disable Run button while running**

In the script, modify `runScenario()`:

Find `running = true; reset(); clearLog();` and immediately after, add:
```javascript
document.getElementById("btn-run").disabled = true;
document.getElementById("btn-run").style.opacity = "0.5";
```

Find `running = false;` at the bottom of `runScenario()` and add immediately after:
```javascript
document.getElementById("btn-run").disabled = false;
document.getElementById("btn-run").style.opacity = "1";
```

- [ ] **Step 2: Add a footer credit line under the canvas**

In the HTML body, find:
```html
      <div id="log" class="log"><div class="line">[idle] click "Run scenario" to start</div></div>
    </div>
  </div>
</div>
```

Replace with:
```html
      <div id="log" class="log"><div class="line">[idle] click "Run scenario" to start</div></div>

      <div style="margin-top: 14px; color: var(--fg-3); font-size: 12px;">
        Inspired by <a style="color: var(--purple);" href="https://ishalkin.github.io/czechitas-pixel-agent/">Czechitas Pixel Agent</a>.
        Brand rules from the <code>accenture-pptx</code> skill (5 rule groups shown of ~30 total).
        Toggle the layers to see what each one adds.
      </div>
    </div>
  </div>
</div>
```

- [ ] **Step 3: Add a hint for first-time visitors below the toggles**

In the HTML, find the rail panel:
```html
      <div class="panel">
        <h3>Layers</h3>
```

After the closing `</div>` of that panel, add a second panel:
```html
      <div class="panel">
        <h3>How to read it</h3>
        <p style="font-size: 12px; color: var(--fg-2); line-height: 1.5;">
          Hit <strong>Run</strong>. Watch the agent walk through 5 stations,
          read the brand skill, call MCP servers, build a slide,
          then pass it through QA. Toggle layers OFF to see what breaks.
        </p>
      </div>
```

- [ ] **Step 4: Verify in browser**

- All toggles render correctly
- Run button greys out during scenario, returns to active when done
- Footer line visible under log
- "How to read it" panel visible under Layers panel

- [ ] **Step 5: Commit**

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Polish: disable Run button while running, add footer + hint panel"
```

---

## Task 10: Acceptance pass

**Files:**
- None modified.

Verify the spec's acceptance criteria.

- [ ] **Step 1: Run all 8 toggle combinations**

Manually:
1. All ON → slide looks branded, QA passes
2. Skill OFF → slide off-brand, QA flags Calibri + 8pt floor
3. MCP OFF → missing logo placeholder, QA flags missing asset
4. Evals OFF → broken slide ships (if Skill+MCP also off), client complains in log
5. Skill+MCP OFF → multiple violations
6. Skill+Evals OFF → broken slide visibly off-brand, no QA gate
7. MCP+Evals OFF → missing assets, no gate
8. All OFF → totally broken, client unhappy

For each, confirm: agent moves smoothly, callouts appear, slide layers render, log lines correct color.

- [ ] **Step 2: First-impression test**

Open in a fresh browser, don't read any text, just hit Run. Within 30 seconds, should be possible to articulate: "agent reads brand rules, calls helper tools for assets, builds slide, gets reviewed."

If unclear → revise copy in the hint panel and station labels.

- [ ] **Step 3: Mobile/narrow check (optional)**

Resize browser to ~700px wide. Layout should stay usable (canvas may scroll horizontally but rail moves above). If broken, add this CSS to the `<style>` block:
```css
@media (max-width: 800px) {
  .layout { grid-template-columns: 1fr; }
  canvas { max-width: 100%; height: auto; }
}
```
Re-verify, commit.

- [ ] **Step 4: Commit any fixes**

```bash
git add pixel-consultant.html
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" commit -m "Acceptance pass fixes" || echo "no changes to commit"
```

---

## Task 11: Deploy to GitHub Pages

**Files:**
- Create: `.github/workflows/pages.yml`

- [ ] **Step 1: Create the GitHub Actions Pages workflow**

Path: `C:/Users/shalkin/czech/pixel-consultant-agent/.github/workflows/pages.yml`

```yaml
name: Deploy to GitHub Pages
on:
  push:
    branches: [main]
permissions:
  contents: read
  pages: write
  id-token: write
concurrency:
  group: "pages"
  cancel-in-progress: true
jobs:
  deploy:
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/configure-pages@v5
      - name: Copy index
        run: cp pixel-consultant.html index.html
      - uses: actions/upload-pages-artifact@v3
        with:
          path: '.'
      - id: deployment
        uses: actions/deploy-pages@v4
```

The `cp pixel-consultant.html index.html` step ensures GitHub Pages serves the file at the repo root URL.

- [ ] **Step 2: Create the GitHub repo via API and push**

```bash
TOKEN=$(cat ~/.gh-ishalkin-token)
curl -s --cacert /c/Users/shalkin/corp-ca-bundle.pem \
  -X POST -H "Authorization: token $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"name":"pixel-consultant-agent","description":"Pixel-art simulation showing how an AI agent uses an Accenture brand skill, MCP servers, and a QA evaluator to build one slide","private":false}' \
  https://api.github.com/user/repos | grep -E '"(full_name|html_url)"' | head -3 | sed 's/ghp_[A-Za-z0-9_]*/[REDACTED]/g'
```

- [ ] **Step 3: Push everything**

```bash
cd /c/Users/shalkin/czech/pixel-consultant-agent
TOKEN=$(cat ~/.gh-ishalkin-token)
BASIC=$(printf "ishalkin:%s" "$TOKEN" | base64 -w0)
git add .github/workflows/pages.yml
git -c user.name="Ilia Shalkin" -c user.email="ishalkin@users.noreply.github.com" \
  commit -m "Add GitHub Pages deploy workflow"
git -c "http.https://github.com/.extraHeader=Authorization: Basic $BASIC" \
  remote add origin https://github.com/IShalkin/pixel-consultant-agent.git
git -c "http.https://github.com/.extraHeader=Authorization: Basic $BASIC" \
  push -u origin main 2>&1 | sed 's/ghp_[A-Za-z0-9_]*/[REDACTED]/g' | tail -5
```

- [ ] **Step 4: Enable Pages source = GitHub Actions**

```bash
TOKEN=$(cat ~/.gh-ishalkin-token)
curl -s --cacert /c/Users/shalkin/corp-ca-bundle.pem \
  -X POST -H "Authorization: token $TOKEN" \
  -H "Content-Type: application/json" \
  -d '{"build_type":"workflow"}' \
  https://api.github.com/repos/IShalkin/pixel-consultant-agent/pages | head -5 | sed 's/ghp_[A-Za-z0-9_]*/[REDACTED]/g'
```

- [ ] **Step 5: Wait for deploy + verify live URL**

```bash
sleep 60
curl -s --cacert /c/Users/shalkin/corp-ca-bundle.pem -o /dev/null -w "HTTP %{http_code}\n" \
  https://ishalkin.github.io/pixel-consultant-agent/
```

Expected: `HTTP 200`. If 404, wait another minute and retry. If still failing, check Action runs:
```bash
TOKEN=$(cat ~/.gh-ishalkin-token)
curl -s --cacert /c/Users/shalkin/corp-ca-bundle.pem \
  -H "Authorization: token $TOKEN" \
  https://api.github.com/repos/IShalkin/pixel-consultant-agent/actions/runs \
  | grep -E '"(status|conclusion|html_url)"' | head -6 | sed 's/ghp_[A-Za-z0-9_]*/[REDACTED]/g'
```

- [ ] **Step 6: Smoke test the live URL**

Open `https://ishalkin.github.io/pixel-consultant-agent/` in a browser. Hit Run. Expected:
- Same behavior as local file
- Agent walks, callouts appear, slide builds, QA passes (with all toggles ON)

---

## Self-review

**Spec coverage:**
- 5 stations (Brief / Skill / MCP / Build / QA) → Tasks 3, 7
- One hero slide layer-by-layer → Tasks 6, 7
- 3 toggles → Tasks 7, 8
- 5 rule groups on Skill station → Task 7 (skill checklist items)
- Pixel-art canvas 480×280, no external assets → Tasks 2-4
- Single file `pixel-consultant.html` → Task 1
- English UI text → all task copy is English
- Accenture brand palette → Task 2 (PAL constants)
- GitHub Pages deploy → Task 11
- 30-second comprehension goal → Task 10 acceptance
- "what does the layer add" demo via toggles → Task 8

**Placeholder scan:** No "TBD", no "implement later", no "similar to Task N" — every step has full code or full command.

**Type/symbol consistency:** `STATIONS`, `agent`, `callouts`, `slide`, `SLIDE_BOX`, `toggles`, `walkTo`, `render`, `runScenario`, `reset`, `drawCallouts`, `drawSlideLayers`, `drawQACircle`, `clearLog`, `log` — all defined before first use, all consistent across tasks.

Plan complete and saved to `C:\Users\shalkin\czech\pixel-consultant-agent\docs\superpowers\plans\2026-05-28-pixel-consultant-agent.md`.
