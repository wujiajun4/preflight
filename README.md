<p align="center">
  <img src="assets/icon.svg" width="80" alt="Preflight" />
</p>

<h1 align="center">✈️ Preflight</h1>

<p align="center">
  <strong>Don't ship half-baked. One command, six checks, pass-or-fail table.<br/>别把半成品推上线。一条命令，六项检查，通过/失败一目了然。</strong>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/version-1.0.0-blue" />
  <img src="https://img.shields.io/badge/platform-Claude%20Code%20%7C%20Agent%20Skills%20%7C%2067%2B-purple" />
  <img src="https://img.shields.io/badge/license-MIT-green" />
  <img src="https://img.shields.io/badge/skills.sh-available-orange" />
  <img src="https://img.shields.io/badge/lang-EN%20%7C%20%E4%B8%AD%E6%96%87-brightgreen" />
</p>

---

> **Every embarrassing "YOUR_USERNAME in the README" bug report is preventable in 3 seconds.** Preflight runs six automated checks before you push. Ship clean. Every time.
>
> **每个尴尬的"README 里写着 YOUR_USERNAME"的 bug 报告，3 秒就能避免。** Preflight 在你 push 前跑六项自动检查。每次发布都干干净净。

---

## 痛点 / The Problem

你写了 3 小时代码。功能跑通了。`git push`。五分钟后：

- 有人开 Issue："你的 README 还写着 `YOUR_USERNAME`"
- 又一个 Issue："有 LICENSE 文件吗？我能用这个吗？"
- 第三个："这个 `ctx_fetch_and_index` 工具我机器上不存在"

**每一个都是 3 秒内能避免的。** 你只是没检查。

没有现有工具解决这个问题。CI/CD 管线检查你的代码——不检查 README。Linter 检查你的语法——不检查文件结构。安全扫描器检查你的依赖——不检查你是否忘了写 `.gitignore`。

---

You've been coding for 3 hours. The feature works. You `git push`. Five minutes later:

- Someone opens an issue: "Your README still says `YOUR_USERNAME`"
- Another issue: "Where's the LICENSE file? Can I even use this?"
- A third: "This `ctx_fetch_and_index` tool doesn't exist on my machine"

**Every one of these is preventable in 3 seconds.** You just didn't check.

No existing tool solves this. CI/CD checks your code — not your README. Linters check your syntax — not your file structure. Security scanners check your dependencies — not whether you forgot a `.gitignore`.

---

## 解决方案 / The Solution

`/preflight` 扫描你的项目，在 push 前跑六项自动检查：

```
/preflight
```

### Demo

```
## Preflight Report — tool-eval

| # | 检查项 Check  | 结果 Result | 详情 Detail                          |
|---|--------------|-----------|-------------------------------------|
| 1 | 占位符 Placeholders | ✅ PASS   | 无 YOUR_USERNAME/TODO/FIXME        |
| 2 | LICENSE      | ❌ FAIL   | LICENSE 文件缺失                   |
| 3 | 私有工具 Private tools | ❌ FAIL   | ctx_fetch_and_index in SKILL.md:37 |
| 4 | .gitignore   | ✅ PASS   | 存在，覆盖 .DS_Store               |
| 5 | 双语 Bilingual | ⚠️ WARN   | 纯英文 README                      |
| 6 | 文件结构 File structure | ✅ PASS  | 全部必需文件齐备                    |

**结论 Verdict: 3/6 通过，2 失败，1 警告。修复 #2 和 #3 后再发布。**
```

三种结果 / Three outcomes:

| 全部 ✅ / All ✅ | 有 ❌ / Has ❌ | 仅 ⚠️ |
|----------------|-------------|-------|
| 发布！/ Ship it | 修复再推 / Fix first | 注明后发布 / Push with caveat |

---

## 六项检查 / The Six Checks

| # | 检查 Check | 怎么查 Command | 为什么重要 Why |
|---|-----------|---------------|---------------|
| 1 | **占位符 Placeholders** | `grep -rn 'YOUR_USERNAME\|TODO\|FIXME' .` | 公开代码里的占位符 = 尴尬 |
| 2 | **LICENSE** | `ls LICENSE` | 声明 MIT 却不给文件 = 法律上模糊 |
| 3 | **私有工具 Private tools** | `grep -rn 'ctx_fetch\|ghost_os' .` | 这些只在你的机器上能跑 |
| 4 | **`.gitignore`** | `ls .gitignore` | `.DS_Store`、`*.log` 泄露到仓库里 |
| 5 | **双语 Bilingual** | 词数分析 / Word-count | 纯英文丢一半受众；纯中文丢另一半 |
| 6 | **文件结构 Structure** | 检查 README + SKILL + LICENSE + .gitignore | 完整的包才叫发布 |

---

## 功能 / Features

| 功能 Feature | 做什么 What it does |
|-------------|-------------------|
| 🔍 **六项检查 / Six checks** | 占位符、LICENSE、私有工具、.gitignore、双语、文件结构 |
| 📊 **成绩单 / Report card** | 通过/失败表格，3 秒读懂 |
| 🔧 **`--fix` 模式** | 自动生成 LICENSE、创建 .gitignore、替换 YOUR_USERNAME |
| 🧪 **`--strict` 严格模式** | 加 4 项检查：死链、行尾空格、图片路径、CHANGELOG |
| 🌐 **双语 / Bilingual** | 中英文报告 |
| 🪶 **零依赖 / Zero deps** | 纯 Bash + grep + ls。无需安装、无运行时开销。 |

---

## 原理 / How It Works

```
/preflight
    │
    ▼
第一步：检测 / Detect ───── 这是 Skill？npm 包？Rust crate？适配检查规则。
    │
    ▼
第二步：扫描 / Scan ────── 一个 Bash 调用跑完 6 项检查
    │
    ▼
第三步：报告 / Report ──── 通过/失败表格 + 结论行
    │
    ▼
第四步（可选）：修复 / Fix ─── 如果传了 --fix，自动修补常见问题
```

---

## 技术栈 / Tech Stack

| 类别 Category | 技术 Technology |
|--------------|----------------|
| 平台 / Platform | Claude Code、OpenClaw、Agent Skills（67+ 平台） |
| 语言 / Language | Bash + Markdown (SKILL.md) |
| 工具 / Tools | grep、ls、wc、find —— 标准 Unix，无花哨 |
| 依赖 / Dependencies | 零 —— 纯 Skill，任何 macOS/Linux 即跑 |

---

## 项目结构 / Project Structure

```
preflight/
├── assets/
│   └── icon.svg          # 剪贴板 + 打勾 + 起飞箭头
├── .gitignore
├── LICENSE               # MIT
├── README.md             # ← 你在这里
└── SKILL.md              # 运行时规范：4 步流水线 + 6 项检查 + 输出模板
```

---

## 模式 / Modes

| 命令 Command | 做什么 What |
|-------------|------------|
| `/preflight` | 跑全部 6 项检查，显示成绩单 |
| `/preflight --strict` | 加 4 项额外检查（死链、行尾空格、图片路径、CHANGELOG） |
| `/preflight --fix` | 自动修复常见问题（生成 LICENSE、创建 .gitignore、替换 YOUR_USERNAME） |

---

## 安装 / Install

```bash
# Claude Code（一行命令 / one command）
mkdir -p ~/.claude/skills && git clone https://github.com/wujiajun4/preflight.git ~/.claude/skills/preflight

# Agent Skills（全平台通用 / any platform）
npx skills add wujiajun4/preflight -g
```

---

## 跟 Tool Eval 配对 / Pair with Tool Eval

```
/preflight          ← 发布前检查自己的项目
/tool-eval <url>    ← 安装前评估别人的工具
```

一个管自己的质量。一个管别人的选择。合在一起，你永远不会发布翻车或安装翻车。

One checks your work. One checks theirs. Together, you never ship broken or install wrong.

---

## FAQ

**Q: 这替代 CI/CD 吗？ / Does this replace CI/CD?**
不。CI/CD 检查代码。Preflight 检查内容。它们是互补层。

**Q: 能加自定义检查吗？ / Can I add custom checks?**
能。编辑 SKILL.md，在 Step 2 的 Bash 块里加检查即可。所有标准 Unix 命令都行。

**Q: 我的项目不需要 LICENSE 怎么办？ / What if my project doesn't need a LICENSE?**
Preflight 会自适应。私有仓库跳过 LICENSE 检查。没有 SKILL.md 的项目跳过文件结构检查。第一步的检测决定哪些规则适用。

**Q: 跟 tool-eval 有什么区别？ / How is this different from tool-eval?**
Tool Eval 评估**别人的**工具。Preflight 检查**自己的**项目。兄弟技能，互补关系。

---

## 贡献 / Contributing

提 Issue 和 PR 都欢迎。提交前——吃自己的狗粮：

```bash
/preflight --strict
```

---

## 许可证 / License

MIT — © 2026 wujiajun4
