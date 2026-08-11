# Lexie Mobile and Desktop Coexistence Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Publish separate mobile operation/result pages alongside the existing desktop operation/result pages, with one homepage that clearly routes to all four experiences.

**Architecture:** Keep each experience as a standalone static HTML file in the GitHub Pages repository. Restore the original mobile operation page byte-for-byte from commit `8a2ed93`, derive a dedicated 480px mobile result page from the existing result content, preserve the current desktop files and URLs, and use `index.html` only as a version selector.

**Tech Stack:** Static HTML/CSS/JavaScript, browser `localStorage`, Node.js syntax/link verification, Python static HTTP server, Git, GitHub CLI, GitHub Pages.

## Global Constraints

- The public site must expose four independent experience pages: mobile operation, mobile result, desktop operation, and desktop result.
- `demo.html` must retain SHA-256 `c28a6c7fa55ffe6aebbadd6a7368d74499d9b45f94a5aff274cb229620577ad6`.
- `result.html` must retain SHA-256 `1ca0e34d3e70f783720f77b7a6b1c69e06d22ed589d8258428148e251388e089`.
- The original mobile operation page must be restored from commit `8a2ed93` rather than approximated by narrowing the desktop page.
- The mobile result page must be a standalone HTML file with a maximum content width of 480px and single-column sections.
- Do not add login, cloud sync, a backend database, or a real volunteer-review system.
- Do not delete or rename the existing desktop pages.

---

## File Structure

- Create `mobile-demo.html`: exact original mobile operation experience from commit `8a2ed93`.
- Create `mobile-result.html`: standalone mobile result experience with the same evidence/content sections as `result.html`.
- Modify `index.html`: version selector with two device groups and four direct links.
- Preserve `demo.html`: desktop operation experience and existing URL.
- Preserve `result.html`: desktop result experience and existing URL.

### Task 1: Restore the Standalone Mobile Operation Page

**Files:**
- Create: `mobile-demo.html`
- Preserve: `demo.html`
- Preserve: `result.html`

**Interfaces:**
- Consumes: Git object `8a2ed93:index.html`.
- Produces: `/mobile-demo.html`, a standalone mobile operation page with the original `localStorage` flow.

- [ ] **Step 1: Confirm the mobile page is not already present**

Run:

```bash
test -f mobile-demo.html
```

Expected: exit code 1 before implementation.

- [ ] **Step 2: Restore the exact original mobile HTML**

Run:

```bash
git show 8a2ed93:index.html > /private/tmp/lexie-mobile-demo-original.html
cp /private/tmp/lexie-mobile-demo-original.html mobile-demo.html
```

- [ ] **Step 3: Verify the restored page is exact and executable**

Run:

```bash
cmp /private/tmp/lexie-mobile-demo-original.html mobile-demo.html
node -e 'const fs=require("fs");const h=fs.readFileSync("mobile-demo.html","utf8");const m=h.match(/<script>([\s\S]*?)<\/script>/);if(!m)throw new Error("missing script");new Function(m[1]);for(const token of ["width:min(100%,480px)","id=\"startBtn\"","localStorage","</html>"]){if(!h.includes(token))throw new Error(`missing ${token}`)}console.log("mobile-demo OK")'
shasum -a 256 demo.html result.html
```

Expected: `cmp` exits 0, Node prints `mobile-demo OK`, and desktop hashes match the global constraints.

- [ ] **Step 4: Commit the restored mobile page**

```bash
git add mobile-demo.html
git commit -m "Restore standalone mobile demo"
```

### Task 2: Add the Standalone Mobile Result Page

**Files:**
- Create: `mobile-result.html`
- Preserve: `result.html`

**Interfaces:**
- Consumes: the complete result content and section IDs from `result.html`.
- Produces: `/mobile-result.html`, a 480px single-column result page with `book`, `gates`, `asm`, `article`, and `feedback` sections.

- [ ] **Step 1: Confirm the mobile result page is not already present**

Run:

```bash
test -f mobile-result.html
```

Expected: exit code 1 before implementation.

- [ ] **Step 2: Copy the complete result content into an independent file**

Run:

```bash
cp result.html mobile-result.html
```

- [ ] **Step 3: Apply the dedicated mobile title and layout overrides**

Change the title to:

```html
<title>每日一小步｜完成歷程與文章結果（手機版範例）</title>
```

Insert this block immediately before `</head>` in `mobile-result.html`:

```html
<style id="dedicated-mobile-layout">
html{background:var(--canvas)}
body{width:min(100%,480px);min-height:100vh;margin:0 auto;background:var(--canvas)}
.topbar-inner{width:100%;padding:10px 14px;gap:8px}
.crumb{display:none}
.score-pill,.done-pill{padding:5px 9px;font-size:11px}
.nav{top:53px}
.nav-inner{width:100%;padding:8px 12px;gap:6px}
.nav a{padding:6px 11px;font-size:12px}
.wrap{width:100%;padding:24px 16px 48px}
h1{font-size:27px}
h2{font-size:20px}
.card{padding:18px}
.hero-grid,.gates-grid,.asm-grid,.article-wrap,.fb-grid{grid-template-columns:1fr;gap:12px}
.stat-grid{gap:7px}
.stat{padding:11px 5px}
.stat .n{font-size:20px}
.gate{padding:15px}
.article{font-size:15px;line-height:1.85}
@media (min-width:481px){body{border-left:1px solid var(--line);border-right:1px solid var(--line)}}
</style>
```

- [ ] **Step 4: Verify the dedicated mobile result contract**

Run:

```bash
node -e 'const fs=require("fs");const h=fs.readFileSync("mobile-result.html","utf8");for(const token of ["id=\"dedicated-mobile-layout\"","width:min(100%,480px)","id=\"book\"","id=\"gates\"","id=\"asm\"","id=\"article\"","id=\"feedback\"","</html>"]){if(!h.includes(token))throw new Error(`missing ${token}`)}console.log("mobile-result OK")'
shasum -a 256 result.html
```

Expected: Node prints `mobile-result OK`; `result.html` still matches the global hash.

- [ ] **Step 5: Commit the mobile result page**

```bash
git add mobile-result.html
git commit -m "Add standalone mobile result"
```

### Task 3: Turn the Homepage Into a Four-Route Version Selector

**Files:**
- Modify: `index.html`
- Test: all five root HTML files through Node link checks.

**Interfaces:**
- Consumes: `mobile-demo.html`, `mobile-result.html`, `demo.html`, and `result.html`.
- Produces: `/`, a homepage with exactly four local experience links grouped under `手機版` and `電腦版`.

- [ ] **Step 1: Run the four-link check before editing**

Run:

```bash
node -e 'const fs=require("fs");const h=fs.readFileSync("index.html","utf8");for(const u of ["mobile-demo.html","mobile-result.html","demo.html","result.html"]){if(!h.includes(`href="${u}"`))throw new Error(`missing ${u}`)}'
```

Expected: FAIL on `mobile-demo.html` before the homepage change.

- [ ] **Step 2: Add grouping styles**

Add these rules to the existing homepage `<style>` block:

```css
.device-group{margin-top:22px}
.device-head{display:flex;align-items:center;gap:10px;margin-bottom:10px}
.device-icon{display:grid;place-items:center;width:38px;height:38px;border-radius:11px;background:var(--blue-soft);font-size:20px}
.device-title{font-size:18px;font-weight:800}
.device-note{font-size:12px;color:var(--muted)}
.link-grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
.link-grid .card{margin-bottom:0}
@media (max-width:560px){.link-grid{grid-template-columns:1fr}}
```

- [ ] **Step 3: Replace the two old cards with two device groups**

Use this body structure after the introductory paragraph:

```html
<section class="device-group" aria-labelledby="mobile-version">
  <div class="device-head">
    <div class="device-icon" aria-hidden="true">📱</div>
    <div><div class="device-title" id="mobile-version">手機版</div><div class="device-note">適合手機直向操作與閱讀</div></div>
  </div>
  <div class="link-grid">
    <a class="card" href="mobile-demo.html"><div class="no">手機 01</div><div class="t">開始操作</div><div class="d">在手機上完成七個閱讀思考關卡。</div><div class="go">開啟手機操作版 →</div></a>
    <a class="card" href="mobile-result.html"><div class="no">手機 02</div><div class="t">查看結果</div><div class="d">查看手機版的完成歷程、文章與回饋。</div><div class="go">開啟手機結果版 →</div></a>
  </div>
</section>
<section class="device-group" aria-labelledby="desktop-version">
  <div class="device-head">
    <div class="device-icon" aria-hidden="true">🖥️</div>
    <div><div class="device-title" id="desktop-version">電腦版</div><div class="device-note">適合桌上型或筆記型電腦寬畫面</div></div>
  </div>
  <div class="link-grid">
    <a class="card" href="demo.html"><div class="no">電腦 01</div><div class="t">開始操作</div><div class="d">使用寬版介面完成七個閱讀思考關卡。</div><div class="go">開啟電腦操作版 →</div></a>
    <a class="card" href="result.html"><div class="no">電腦 02</div><div class="t">查看結果</div><div class="d">查看電腦版的完成歷程、文章與回饋。</div><div class="go">開啟電腦結果版 →</div></a>
  </div>
</section>
```

- [ ] **Step 4: Verify all routes and preserve desktop hashes**

Run:

```bash
node -e 'const fs=require("fs"),path=require("path");const files=["index.html","mobile-demo.html","mobile-result.html","demo.html","result.html"];for(const name of files){const h=fs.readFileSync(name,"utf8");if(!/<\/html>\s*$/.test(h))throw new Error(`${name}: incomplete`);for(const m of h.matchAll(/<script(?:\s[^>]*)?>([\s\S]*?)<\/script>/g))new Function(m[1]);for(const m of h.matchAll(/(?:href|src)="([^"]+)"/g)){const u=m[1];if(!u.startsWith("#")&&!/^(https?:|data:|mailto:|tel:)/.test(u)&&!fs.existsSync(path.join(".",u.split("#")[0])))throw new Error(`${name}: missing ${u}`)}}console.log("five-page verification OK")'
shasum -a 256 demo.html result.html
git diff --check
```

Expected: `five-page verification OK`; both desktop hashes match the global constraints; `git diff --check` is empty.

- [ ] **Step 5: Commit the homepage selector**

```bash
git add index.html
git commit -m "Add mobile and desktop version selector"
```

### Task 4: Validate Locally, Publish, and Verify GitHub Pages

**Files:**
- Verify: `index.html`
- Verify: `mobile-demo.html`
- Verify: `mobile-result.html`
- Verify: `demo.html`
- Verify: `result.html`

**Interfaces:**
- Consumes: the five static pages produced and preserved by Tasks 1–3.
- Produces: five HTTP 200 public routes under `https://azt156.github.io/lexie-mobile-demo/` and a clean `main` branch tracking `origin/main`.

- [ ] **Step 1: Serve the site locally**

Run from the repository root:

```bash
python3 -m http.server 4275 --bind 127.0.0.1
```

Expected: server listens on `127.0.0.1:4275`.

- [ ] **Step 2: Fetch every local route and compare files**

Run in another shell:

```bash
curl -fsS http://127.0.0.1:4275/ -o /private/tmp/lexie-index.html
curl -fsS http://127.0.0.1:4275/mobile-demo.html -o /private/tmp/lexie-mobile-demo.html
curl -fsS http://127.0.0.1:4275/mobile-result.html -o /private/tmp/lexie-mobile-result.html
curl -fsS http://127.0.0.1:4275/demo.html -o /private/tmp/lexie-desktop-demo.html
curl -fsS http://127.0.0.1:4275/result.html -o /private/tmp/lexie-desktop-result.html
cmp index.html /private/tmp/lexie-index.html
cmp mobile-demo.html /private/tmp/lexie-mobile-demo.html
cmp mobile-result.html /private/tmp/lexie-mobile-result.html
cmp demo.html /private/tmp/lexie-desktop-demo.html
cmp result.html /private/tmp/lexie-desktop-result.html
```

Expected: all `curl` commands exit 0 and every `cmp` exits 0.

- [ ] **Step 3: Run the final repository verification**

Run:

```bash
git status -sb
git log --oneline -5
node -e 'const fs=require("fs");const h=fs.readFileSync("index.html","utf8");const links=[...h.matchAll(/href="([^"]+\.html)"/g)].map(m=>m[1]);if(JSON.stringify(links)!==JSON.stringify(["mobile-demo.html","mobile-result.html","demo.html","result.html"]))throw new Error(JSON.stringify(links));console.log("homepage route order OK")'
```

Expected: clean working tree ahead of `origin/main`; route order is mobile operation, mobile result, desktop operation, desktop result.

- [ ] **Step 4: Push the approved implementation**

Run:

```bash
git push origin main
```

Expected: `main -> main` with no rejected updates.

- [ ] **Step 5: Wait for the exact GitHub Pages commit**

Run:

```bash
git rev-parse HEAD
gh api repos/azt156/lexie-mobile-demo/pages/builds/latest --jq '{status,error,commit,updated_at}'
```

Expected: `status` becomes `built`, `error.message` is null, and `commit` equals local `HEAD`.

- [ ] **Step 6: Fetch and compare all public routes**

Run:

```bash
curl -fsSL https://azt156.github.io/lexie-mobile-demo/ -o /private/tmp/lexie-live-index.html
curl -fsSL https://azt156.github.io/lexie-mobile-demo/mobile-demo.html -o /private/tmp/lexie-live-mobile-demo.html
curl -fsSL https://azt156.github.io/lexie-mobile-demo/mobile-result.html -o /private/tmp/lexie-live-mobile-result.html
curl -fsSL https://azt156.github.io/lexie-mobile-demo/demo.html -o /private/tmp/lexie-live-desktop-demo.html
curl -fsSL https://azt156.github.io/lexie-mobile-demo/result.html -o /private/tmp/lexie-live-desktop-result.html
cmp index.html /private/tmp/lexie-live-index.html
cmp mobile-demo.html /private/tmp/lexie-live-mobile-demo.html
cmp mobile-result.html /private/tmp/lexie-live-mobile-result.html
cmp demo.html /private/tmp/lexie-live-desktop-demo.html
cmp result.html /private/tmp/lexie-live-desktop-result.html
git status -sb
```

Expected: all five public files match the repository byte-for-byte, and the branch is clean and synchronized with `origin/main`.
