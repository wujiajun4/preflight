---
name: preflight
version: "2.0.0"
description: >-
  L1: 6 content checks + 3 distribution checks. Don't just write good code — make sure people can find it. L2: full pipeline with --strict, --fix, and --distribute modes. | 中文触发：AI 工具。 Use this skill when the user mentions tool / skill / utility / AI 助手 / agent tool.
allowed-tools: Bash, Read, Write, Glob, Grep
user-invocable: true
tags: [release, checklist, quality, publishing, github, pre-commit, distribution, growth]
argument-hint: "preflight | preflight --strict | preflight --fix | preflight --distribute"

progressive_disclosure:
  enabled: true
  level1_tokens: 100
  level2_tokens: 3500

triggers:
  keywords:
    - "preflight"
    - "发布前"
    - "能发了吗"
    - "check before push"
    - "发布前检查"
  events:
    - "pre-push"
    - "pre-release"
  phases:
    - "placeholders"
    - "license"
    - "private-tools"
    - "gitignore"
    - "bilingual"
    - "file-structure"
    - "github-topics"
    - "skills-sh"
    - "install-visibility"
---

<!-- LEVEL 1 — Read this first -->

| Mode | Checks | Use when |
|------|--------|----------|
| `preflight` | 6 content checks | Every release |
| `preflight --distribute` | 6 content + 3 distribution | Publishing a skill/product |
| `preflight --strict` | 10 content checks | Polishing pass |
| `preflight --fix` | Auto-fix common issues | Quick cleanup |

<!-- LEVEL 2 — Full pipeline starts here -->

# Preflight — "Don't ship half-baked. Don't ship invisible."

## The Problem (v2.0 insight)

v1.0 caught content bugs — placeholders, missing LICENSE, private tools. But after pushing
4 products and getting **zero organic discovery**, the real gap became clear:

**Content quality ≠ distribution quality.**

You can have a perfect README, but if your repo has no topics, it's invisible to GitHub search.
If it's not on skills.sh, the 67-platform skill ecosystem can't find it. If the install command
appears once buried in the README, nobody knows how to use it.

## Modes

| Mode | Checks | When |
|------|--------|------|
| **Default** (`preflight`) | #1–6 Content | Every release |
| **Distribute** (`preflight --distribute`) | #1–6 Content + #7–9 Distribution | Publishing a skill or public product |
| **Strict** (`preflight --strict`) | #1–10 All | Final polish pass |
| **Fix** (`preflight --fix`) | #1–6 + auto-patch | Quick cleanup |

## The Nine Checks

### Content Checks (#1–6)

| # | Check | Why it matters |
|---|-------|---------------|
| 1 | **Placeholders** | `YOUR_USERNAME`, `TODO`, `FIXME` in published code is embarrassing |
| 2 | **LICENSE file** | Declaring MIT without a `LICENSE` file = legally ambiguous |
| 3 | **No private tool refs** | `ctx_fetch_and_index`, `ctx_execute` only work on your machine |
| 4 | **`.gitignore` exists** | Without it, `.DS_Store` and logs leak into the repo |
| 5 | **Bilingual README** | English-only misses half your audience; Chinese-only misses the other half |
| 6 | **File structure** | Missing README? Missing SKILL.md? Ship the full package |

### Distribution Checks (#7–9) — only in `--distribute` mode

| # | Check | Why it matters | How to verify |
|---|-------|---------------|---------------|
| 7 | **GitHub Topics** | Empty topics = invisible to GitHub search | `gh api repos/{owner}/{repo}/topics` → must have ≥5 |
| 8 | **skills.sh discoverable** | This is the 67-platform skill marketplace | Check: repo has `skill` + `agent-skills` topics AND a `SKILL.md` with `user-invocable: true` |
| 9 | **Install command visibility** | Users need to see the command 3+ times to act | Count `npx skills add` or `git clone` occurrences in README.md → should appear ≥3 times |

## Pipeline

### Step 1: Detect Project Type
```bash
ls README.md 2>/dev/null && echo "HAS_README" || echo "NO_README"
ls SKILL.md 2>/dev/null && echo "HAS_SKILL" || echo "NO_SKILL"
ls package.json 2>/dev/null && echo "NPM" || true
# Extract repo name from git remote for API checks
git remote get-url origin 2>/dev/null | sed 's/.*github.com[:\/]\(.*\)\.git/\1/'
```

### Step 2: Run Content Checks (#1–6)

```bash
echo "=== CHECK 1: PLACEHOLDERS ==="
PLACEHOLDER_FILES=$(grep -rn 'YOUR_USERNAME\|<<<TODO>>>\|FIXME\|XXX_REPLACE_ME' \
  README.md SKILL.md LICENSE CHANGELOG.md package.json 2>/dev/null)
if [ -z "$PLACEHOLDER_FILES" ]; then
  echo "PASS: No placeholder strings found"
else
  echo "FAIL: Placeholders found:"
  echo "$PLACEHOLDER_FILES"
fi

echo "=== CHECK 2: LICENSE ==="
if ls LICENSE* 1>/dev/null 2>&1; then
  echo "PASS: LICENSE file exists"
else
  echo "FAIL: No LICENSE file found"
fi

echo "=== CHECK 3: PRIVATE TOOL REFS ==="
PRIV_TOOLS=$(grep -rn 'ctx_fetch_and_index\|ctx_execute_file\|ctx_batch_execute\|ghost_os\|mcp__ghost-os' \
  SKILL.md README.md 2>/dev/null)
if [ -z "$PRIV_TOOLS" ]; then
  echo "PASS: No private tool references found"
else
  echo "FAIL: Private tool references found:"
  echo "$PRIV_TOOLS"
fi

echo "=== CHECK 4: GITIGNORE ==="
if ls .gitignore 1>/dev/null 2>&1; then
  HAS_DSSTORE=$(grep -c '\.DS_Store' .gitignore 2>/dev/null || echo 0)
  echo "PASS: .gitignore exists (covers .DS_Store: $([ "$HAS_DSSTORE" -gt 0 ] && echo 'yes' || echo 'no'))"
else
  echo "FAIL: No .gitignore file found"
fi

echo "=== CHECK 5: BILINGUAL ==="
README_LINES=$(wc -l < README.md 2>/dev/null || echo 0)
if [ "$README_LINES" -gt 0 ]; then
  HAS_EN=$(grep -c '[A-Za-z]\{20,\}' README.md 2>/dev/null || echo 0)
  HAS_CN=$(grep -c '[\x{4e00}-\x{9fff}]\{4,\}' README.md 2>/dev/null || echo 0)
  echo "README: $README_LINES lines, EN segments: $HAS_EN, CN segments: $HAS_CN"
  if [ "$HAS_EN" -gt 0 ] && [ "$HAS_CN" -gt 0 ]; then
    echo "PASS: README has both English and Chinese content"
  elif [ "$HAS_EN" -gt 0 ]; then
    echo "WARN: README is English-only"
  elif [ "$HAS_CN" -gt 0 ]; then
    echo "WARN: README is Chinese-only"
  fi
else
  echo "SKIP: No README.md found"
fi

echo "=== CHECK 6: FILE STRUCTURE ==="
FILES_OK=true
for f in README.md .gitignore; do
  if [ ! -f "$f" ]; then echo "MISSING: $f"; FILES_OK=false; fi
done
if [ -f "SKILL.md" ]; then
  for f in SKILL.md LICENSE assets/; do
    if [ ! -e "$f" ]; then echo "MISSING (skill): $f"; FILES_OK=false; fi
  done
fi
$FILES_OK && echo "PASS: File structure complete" || echo "FAIL: Missing files detected"
```

### Step 3: Run Distribution Checks (#7–9) — only in `--distribute` mode

These checks need network access. Use `ctx_execute` (Node.js) to query GitHub API:

```javascript
// CHECK 7: GitHub Topics
const resp = await fetch('https://api.github.com/repos/{owner}/{repo}/topics', {
  headers: { 'Accept': 'application/vnd.github+json', 'User-Agent': 'preflight' }
});
const data = await resp.json();
const topics = data.names || [];
if (topics.length >= 5) {
  console.log('PASS: ' + topics.length + ' GitHub topics set: ' + topics.join(', '));
} else if (topics.length > 0) {
  console.log('WARN: Only ' + topics.length + ' topics. Recommend ≥5. Missing: claude-code, skill, agent-skills');
} else {
  console.log('FAIL: ZERO GitHub topics. Repo is invisible to search.');
  console.log('→ Fix: Add topics at github.com/{owner}/{repo}/settings (top of page)');
  console.log('→ Recommended: claude-code, skill, agent-skills, developer-tools, {domain-specific}');
}

// CHECK 8: skills.sh discoverable
// Read SKILL.md frontmatter
const skillContent = require('fs').readFileSync('SKILL.md', 'utf8');
const isUserInvocable = skillContent.includes('user-invocable: true');
const hasSkillTag = topics.some(t => t === 'skill' || t === 'agent-skills');
if (hasSkillTag && isUserInvocable) {
  console.log('PASS: skills.sh will auto-index this skill (topics + SKILL.md ok)');
} else {
  console.log('WARN: May not be auto-discovered by skills.sh');
  if (!hasSkillTag) console.log('→ Add topic: skill or agent-skills');
  if (!isUserInvocable) console.log('→ Set user-invocable: true in SKILL.md frontmatter');
}

// CHECK 9: Install command visibility
const readme = require('fs').readFileSync('README.md', 'utf8');
const installCount = (readme.match(/npx skills add/g) || []).length +
                     (readme.match(/git clone.*github\.com\/{owner}\/{repo}/g) || []).length;
if (installCount >= 3) {
  console.log('PASS: Install command appears ' + installCount + ' times in README');
} else if (installCount > 0) {
  console.log('WARN: Install command only appears ' + installCount + ' times. Recommend ≥3.');
  console.log('→ Add it: below header, in Install section, and in footer');
} else {
  console.log('FAIL: No install command found in README');
  console.log('→ Add: npx skills add {owner}/{repo} -g');
}
```

### Step 4: Output the Report Card

```
## Preflight Report — {project_name}

### Content (#1–6)
| # | Check | Result | Detail |
|---|-------|--------|--------|
| 1 | Placeholders | ✅ PASS | No YOUR_USERNAME/TODO/FIXME found |
| 2 | LICENSE | ✅ PASS | MIT license present |
| 3 | Private tools | ✅ PASS | Clean |
| 4 | .gitignore | ✅ PASS | Exists, covers .DS_Store |
| 5 | Bilingual | ⚠️ WARN | English-only README |
| 6 | File structure | ✅ PASS | README+SKILL+LICENSE+gitignore+assets |

### Distribution (#7–9) [--distribute mode]
| # | Check | Result | Detail |
|---|-------|--------|--------|
| 7 | GitHub Topics | ❌ FAIL | 0 topics set. Repo invisible to GitHub search. |
| 8 | skills.sh | ⚠️ WARN | Missing 'skill' topic. Add it for auto-indexing. |
| 9 | Install visibility | ⚠️ WARN | Install command appears only 1 time. Add 2+ more. |

**Verdict: Content 5/6, Distribution 0/3. Fix #7 before shipping.**
```

### Step 5 (Optional): Auto-Fix Mode (`--fix`)

- Placeholders → replace `YOUR_USERNAME` with detected GitHub user
- LICENSE → generate MIT template with current year
- .gitignore → create with `.DS_Store`, `*.log`, `node_modules/`
- Private tool refs → warn but don't auto-replace (needs human judgment)
- **NEW: README install command** → if missing from header, inject `<code>npx skills add ...</code>` after badge row

## Strict Mode (`--strict`)

Adds #10–13:
10. Markdown links not broken
11. No trailing whitespace
12. All image paths resolve
13. CHANGELOG.md has entry for current version

## Rules

- **Never modifies files unless `--fix` is explicitly passed.**
- **Distribution checks need network.** They call GitHub API. In sandbox, use `ctx_execute` Node.js.
- **Distribution checks are warnings by default in `preflight`, FAILs in `--distribute`.**
- **The report card is the product.** Table + verdict, no walls of text.
- **Failures are specific.** "0 GitHub topics" not "distribution issues."

## v2.0 Changelog

| What | Why |
|------|-----|
| `--distribute` mode | Content quality ≠ distribution quality. 4 shipped products got 0 organic discovery. |
| Check #7: GitHub Topics | Empty topics = invisible to GitHub search. Caught on 2026-06-06 launch. |
| Check #8: skills.sh discoverable | 67-platform ecosystem can't find skills without `skill` topic + SKILL.md. |
| Check #9: Install visibility | Users need 3+ install command sightings to act. Hot repos have 24-26. |
| Progressive disclosure | L1 100-token mode summary for fast routing. |
| Structured triggers | Keywords + events for auto-invoke. |

### v2.0.1 (2026-06-07)

| What | Why |
|------|-----|
| **FIX**: Move `argument-hint` to dedicated frontmatter field | Was crammed in description after 126 SKILL.md bulk rewrite — same regression that hit brain-sync and tool-eval |
| **FIX**: Add CJK trigger inline in description | Standard pattern from skill-lens v1.1.0 |
| **NEW**: Add `triggers.phases` block | Maps 9 checks to execution phases (placeholders, license, private-tools, gitignore, bilingual, file-structure, github-topics, skills-sh, install-visibility) |
| **NEW**: v2.0.1 changelog section | Transparency for patch-level changes |
| **FIX**: git remote URL cleaned | Was leaking old PAT `ghp_TVHOf***` (旧 token 已撤销，仅前 10 字符保留供识别) |

---

## Security Note

This skill is an instruction-only skill (no scripts). For related skills with scripts, see
`lens.cjs` in skill-lens and the secure-push pattern documentation. Key principles:

- **Never write full PAT tokens in documentation** — use `ghp_TVHOf***` masked form
- **Always run `git remote set-url` after push** to clean embedded old PATs
- **GitHub secret scanner watches SKILL.md content** — not just commit messages</mm:think>
