---
name: preflight
version: "1.0.0"
description: >-
  Pre-release checklist for any project. Scans for placeholders, missing LICENSE,
  private tool references, broken .gitignore, bilingual gaps, and incomplete file
  structure. One command, six checks, pass-or-fail table. Never ship half-baked again.
argument-hint: "preflight | preflight --strict | preflight --fix"
allowed-tools: Bash, Read, Write, Glob, Grep
user-invocable: true
tags: [release, checklist, quality, publishing, github, pre-commit]
---

# Preflight — "Don't ship half-baked."

One command. Six automated checks. A pass-or-fail table you can read in 3 seconds.
Never push a broken release again.

## When to use

| Trigger | Example |
|---------|---------|
| Before pushing to GitHub | `preflight` |
| Before publishing a skill | `preflight` |
| Strict mode (also check markdown links, trailing whitespace) | `preflight --strict` |
| Auto-fix common issues | `preflight --fix` |

## The Six Checks

| # | Check | Why it matters |
|---|-------|---------------|
| 1 | **Placeholders** | `YOUR_USERNAME`, `TODO`, `FIXME` in published code is embarrassing |
| 2 | **LICENSE file** | Declaring MIT without a `LICENSE` file = legally ambiguous |
| 3 | **No private tool refs** | `ctx_fetch_and_index`, `ctx_execute` only work on your machine |
| 4 | **`.gitignore` exists** | Without it, `.DS_Store` and logs leak into the repo |
| 5 | **Bilingual README** | English-only misses half your audience; Chinese-only misses the other half |
| 6 | **File structure** | Missing README? Missing SKILL.md? Ship the full package |

## Pipeline

### Step 1: Detect Project Type
```bash
ls README.md 2>/dev/null && echo "HAS_README" || echo "NO_README"
ls SKILL.md 2>/dev/null && echo "HAS_SKILL" || echo "NO_SKILL"
ls package.json 2>/dev/null && echo "NPM" || true
ls Cargo.toml 2>/dev/null && echo "RUST" || true
ls go.mod 2>/dev/null && echo "GO" || true
```
This tells you which checks apply. Not every project needs SKILL.md. Not every project needs a README in Chinese.

### Step 2: Run All Checks

Run these in one `Bash` call for speed:

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
  echo "FAIL: Private tool references found (these only work on your machine):"
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
    echo "WARN: README is English-only — consider adding Chinese examples"
  elif [ "$HAS_CN" -gt 0 ]; then
    echo "WARN: README is Chinese-only — consider adding English examples"
  fi
else
  echo "SKIP: No README.md found"
fi

echo "=== CHECK 6: FILE STRUCTURE ==="
FILES_OK=true
for f in README.md .gitignore; do
  if [ ! -f "$f" ]; then
    echo "MISSING: $f"
    FILES_OK=false
  fi
done
# If SKILL.md was detected in Step 1
if [ -f "SKILL.md" ]; then
  for f in SKILL.md LICENSE assets/; do
    if [ ! -e "$f" ]; then
      echo "MISSING (skill): $f"
      FILES_OK=false
    fi
  done
fi
if $FILES_OK; then
  echo "PASS: File structure complete"
else
  echo "FAIL: Missing files detected"
fi

echo "=== SUMMARY ==="
```

### Step 3: Output the Report Card

Parse the output and present a table:

```
## Preflight Report — {project_name}

| # | Check | Result | Detail |
|---|-------|--------|--------|
| 1 | Placeholders | ✅ PASS | No YOUR_USERNAME/TODO/FIXME found |
| 2 | LICENSE | ✅ PASS | MIT license present |
| 3 | Private tools | ❌ FAIL | ctx_execute found in SKILL.md:42 |
| 4 | .gitignore | ✅ PASS | Exists, covers .DS_Store |
| 5 | Bilingual | ⚠️ WARN | English-only README |
| 6 | File structure | ✅ PASS | All required files present |

**Verdict: 4/6 passed, 1 failed, 1 warning. Fix #3 before shipping.**
```

### Step 4 (Optional): Auto-Fix Mode

If `--fix` flag is passed, attempt automatic fixes:
- Placeholders → replace `YOUR_USERNAME` with detected GitHub user
- LICENSE → generate MIT template with current year
- .gitignore → create with `.DS_Store`, `*.log`, `node_modules/`
- Private tool refs → warn but don't auto-replace (needs human judgment)

## Output Format Rules

- **Always show the table.** The six-row report card is the product.
- **PASS = green check, FAIL = red X, WARN = yellow warning, SKIP = grey dash.**
- **Verdict line at the bottom** with counts and a clear go/no-go.
- **If any FAIL, do not recommend pushing.** Tell the user exactly what to fix.
- **If only WARNs, mention them but allow push with caveats.**

## Strict Mode (`--strict`)

Add these extra checks:
7. Markdown links are not broken (`grep -oP '\[.*?\]\((\K[^)]+)' *.md | while read url; do curl -sI "$url" | head -1; done`)
8. No trailing whitespace (`grep -rn '[[:space:]]$' *.md`)
9. All image paths resolve (`ls assets/*.svg assets/*.png 2>/dev/null`)
10. CHANGELOG.md has an entry for the current version

## Rules

- **Works on any project type.** Detect npm/Rust/Go/skill and adapt checks accordingly.
- **Never modifies files unless `--fix` is explicitly passed.**
- **One Bash call for all checks.** Speed matters. Don't run six separate commands.
- **Failures are specific.** "Missing LICENSE" not "something's wrong with your project."
- **The report card is the product.** No walls of text, just the table + verdict.
