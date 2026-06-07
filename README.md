<div align="center">
  <img src="assets/icon.svg" width="80" alt="Preflight" />
</div>

<h1 align="center">✈️ Preflight v2.0</h1>

<p align="center">
  <strong>Don't ship half-baked. Don't ship invisible.<br/>别把半成品推上线。别让好产品没人发现。</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-2.0.0-blue" />
  <img src="https://img.shields.io/badge/platform-Claude%20Code%20%7C%20Agent%20Skills%20%7C%2067%2B-purple" />
  <img src="https://img.shields.io/badge/license-MIT-green" />
  <img src="https://img.shields.io/badge/skills.sh-available-orange" />
  <img src="https://img.shields.io/badge/lang-EN%20%7C%20%E4%B8%AD%E6%96%87-brightgreen" />
</p>

<p align="center">
  <code>npx skills add wujiajun4/preflight -g</code>
</p>

---

> **v2.0 ships with distribution checks.** v1 caught content bugs (placeholders, missing LICENSE). But 4 shipped products got zero discovery — because nobody checked GitHub topics, skills.sh visibility, or install command presence. v2 closes the gap.
>
> **v2.0 新增分发检查。** v1 只检查内容质量。但今天发布的 4 个产品零发现——因为没人检查 GitHub topics、skills.sh 可见度、安装命令出现次数。v2 补上这个盲区。

---

## 痛点 / The Problem

### v1 查不出的问题

你 `git push` 了。READMe 完美。LICENSE 齐全。然后——零⭐。

不是质量不行，是**没人找得到你**：

| 症状 | 根因 | preflight v1 查了没 |
|------|------|------|
| GitHub 搜索找不到 | repo topics 为空 | ❌ 没查 |
| skills.sh 没收录 | 没加 `skill` topic | ❌ 没查 |
| 用户不知道怎么装 | 安装命令只在 README 里出现 1 次 | ❌ 没查 |

**质量好 ≠ 被发现。** v2 把分发也变成自动化检查。

---

## 模式 / Modes

| 命令 | 检查项 | 什么时候用 |
|------|--------|------|
| `/preflight` | 1–6 内容质量 | 每次发布 |
| `/preflight --distribute` | 1–9 内容+分发 | 发布 Skill/公开产品 |
| `/preflight --strict` | 1–13 全部 | 最终打磨 |
| `/preflight --fix` | 1–6 + 自动修复 | 快速清理 |

---

## 内容检查 / Content Checks (#1–6)

| # | 检查 | 抓什么 |
|---|------|--------|
| 1 | **占位符** | `YOUR_USERNAME`, `TODO`, `FIXME` |
| 2 | **LICENSE** | 声明了 MIT 但没文件 = 法律模糊 |
| 3 | **私有工具** | `ctx_fetch_and_index` 只在你机器上能用 |
| 4 | **.gitignore** | `.DS_Store`, `*.log` 泄露到仓库 |
| 5 | **双语** | 纯英文丢一半受众，纯中文丢另一半 |
| 6 | **文件结构** | README + SKILL + LICENSE + .gitignore 齐全 |

## 分发检查 / Distribution Checks (#7–9) 🆕

| # | 检查 | 抓什么 | 怎么查 |
|---|------|--------|--------|
| 7 | **GitHub Topics** | 0 topics = GitHub 搜索找不到你 | GitHub API → 必须 ≥5 个 |
| 8 | **skills.sh 可见度** | 67 平台生态找不到你的 skill | 检查 `skill` topic + `user-invocable: true` |
| 9 | **安装命令曝光** | 用户看到 1 次不会行动 | 读 README → `npx skills add` 需要 ≥3 次 |

---

## 案例 / Case Study

**2026-06-06: 发布 4 个产品后零发现。** preflight v1 检查全部通过。但：

```
Check #7: GitHub Topics → 0/4 repos had topics → invisible to search
Check #8: skills.sh → no `skill` topic → not auto-indexed
Check #9: Install command → 1-2 occurrences → users didn't know how to install
```

v2 的 `--distribute` 模式就是为这个教训而生的。

---

## 输出 / Output

```
## Preflight Report — tool-eval

### Content (#1–6)
| # | Check | Result | Detail |
|---|-------|--------|--------|
| 1 | Placeholders | ✅ PASS | Clean |
| 2 | LICENSE | ✅ PASS | MIT |
| 3 | Private tools | ✅ PASS | Clean |
| 4 | .gitignore | ✅ PASS | Exists |
| 5 | Bilingual | ✅ PASS | 272 lines, EN+CN |
| 6 | File structure | ✅ PASS | Complete |

### Distribution (#7–9) [--distribute]
| # | Check | Result | Detail |
|---|-------|--------|--------|
| 7 | GitHub Topics | ✅ PASS | 9 topics: claude-code, skill, agent-skills, ... |
| 8 | skills.sh | ✅ PASS | Auto-indexed |
| 9 | Install visibility | ✅ PASS | npx skills add appears 4 times |

**Verdict: 9/9 PASS. Ship it.**
```

---

## 安装 / Install

```bash
npx skills add wujiajun4/preflight -g
```

| 你说 | 发生什么 |
|------|------|
| `/preflight` | 6 项内容检查 |
| `/preflight --distribute` | 9 项全查 |
| `/preflight --strict` | 13 项严格模式 |
| `/preflight --fix` | 自动修复占位符/LICENSE/.gitignore |

---

## 与你其他 Skill 的关系

```
preflight     → 检查自己的项目（质量+分发）
tool-eval     → 评估别人的工具（该不该装）
brain-sync    → 同步大小脑（数据一致）
skill-lens    → 检测盲区（执行覆盖）
```

---

## v2.0 更新日志

| 变更 | 原因 |
|------|------|
| `--distribute` 模式 + 3 项分发检查 | 4 产品零发现 → 分发也需要自动化 |
| 渐进式披露 | 减少无关时的上下文占用 |
| 标准化触发词 | keywords + events 更精准匹配 |

---

## 许可证 / License

MIT — © 2026 wujiajun4
