# Patent-book-to-skill

把三部中国专利法规文本蒸馏成**一个** Agent Skill（[Agent Skills 标准](https://github.com/agentskills/agentskills)），供 Claude Code、GitHub Copilot CLI、Codex、Amp、Hermes、OpenClaw、OpenCode 等支持该标准的编码/办公 agent 按需加载。生成方法是 [book-to-skill](https://github.com/virgiliojr94/book-to-skill)：只提取结构（规则、判断标准、期限、后果、术语），不复制原文。

技能名：[`prc-patent-law-system`](skills/prc-patent-law-system/SKILL.md)。它把同一体系的三层放在一个入口里，一次调用即可同时查三部文本，并在三层之间互相索引：

| 层 | 来源 | 章节 ID | 回答什么问题 |
|---|---|---|---|
| 法 | 《中华人民共和国专利法》2020 年第四次修正（`专利法2020.pdf`，24 页，82 条） | `law-ch01`–`law-ch08` | 法条本身：客体、授权条件、申请与审查、期限与无效、许可、保护与赔偿——"第 N 条怎么规定" |
| 细则 | 《中华人民共和国专利法实施细则》2023 年修订（`专利法实施细则2023.txt`，自国家知识产权局网站抓取，13 章 149 条） | `reg-ch01`–`reg-ch13` | 程序上怎么做、期限多长、逾期后果是什么：手续与送达、期限计算与恢复、申请文件格式、优先权与保藏、受理与申请日、初审/实审/修改/驳回/登记、复审与无效、期限补偿、开放/强制许可、奖励报酬、假冒专利与行政处理、登记公报、费用、PCT 与海牙国际申请 |
| 指南 | 《专利审查指南》2023 年版（`专利审查指南2023.pdf`，613 页，6 部分 38 章） | `gl-p1-ch01`–`gl-p6-ch02` | 审查员按什么标准判断：初审、实审（三性、说明书与权利要求、检索、程序、计算机程序/化学/中药）、复审与无效、PCT 国家阶段、海牙外观设计、事务处理（受理、费用、期限、送达、保密审查、公报、授权与终止、评价报告、开放许可） |

三层如何互相索引：

- `SKILL.md`（常驻，约 4K tokens）：按问题形态路由到某一层，核心规则按专利生命周期组织、每条同时引法条 / 细则条 / 指南章节，并给出三层并列的主题索引。
- `crosswalk.md`：条文对照表——专利法第 N 条 → 所在章节 + 引用该条的细则章节与指南章节；细则第 N 条 → 所在章节 + 引用它的指南章节。由章节文件中的引注自动汇总。
- `index.md`：59 章完整索引（链接 + 每章关键规则）和三层合一的主题索引（227 条，按拼音排序）。
- `glossary.md`：540 条术语（法 51 / 细则 96 / 指南 393），拼音排序并标注来源层，同名术语在三层的定义相邻排列。
- `patterns.md`：43 个程序模式按程序阶段分十组，同一阶段里法 / 细则 / 指南的模式相邻。
- `cheatsheet.md`：期限与后果表每行同时给出法 / 细则 / 指南三列出处，加三类专利对比、数字门槛、决策规则、决策树、失权信号。
- 每个章节文件开头标明来源层与章节 ID，文末「关联」列出另两层的对应章节。

## 安装

技能在 `skills/prc-patent-law-system/`，以 `SKILL.md` 为入口。若仓库为私有，安装时需要能访问它的 GitHub 凭据。

**方式一：skills CLI**

```bash
npx skills add https://github.com/Nick-JuRan/Patent-book-to-skill --skill prc-patent-law-system
```

**方式二：git clone 后链接到 agent 的技能目录**

```bash
git clone https://github.com/Nick-JuRan/Patent-book-to-skill.git
cd Patent-book-to-skill

# 跨 agent 的个人技能目录（Copilot CLI、Amp、Codex、OpenCode 原生发现）
mkdir -p ~/.agents/skills
ln -s "$PWD/skills/prc-patent-law-system" ~/.agents/skills/prc-patent-law-system

# Claude Code 只扫描 ~/.claude/skills，再链一次
mkdir -p ~/.claude/skills
ln -s "$PWD/skills/prc-patent-law-system" ~/.claude/skills/prc-patent-law-system
```

项目级安装：把 `skills/prc-patent-law-system` 复制或链接到项目的 `.claude/skills/`、`.agents/skills/`、`.github/skills/` 或 `.opencode/skills/`。Hermes 用 `$HERMES_HOME/skills/<分类>/prc-patent-law-system`，OpenClaw 用 `${OPENCLAW_STATE_DIR:-~/.openclaw}/skills/prc-patent-law-system`。装好后重启 agent 会话（Copilot CLI 用 `/skills reload`）。

此前的三个单文本技能（`prc-patent-law-2020`、`prc-patent-implementing-regulations-2023`、`prc-patent-examination-guidelines-2023`）已合并进本技能，不再单独维护；需要旧版可在 git 历史中找到。

## 用法

技能是按需加载的：`SKILL.md` 常驻（路由 + 核心规则 + 索引），章节文件和 index / crosswalk / 术语表 / 模式 / 速查表只在需要时读取。

```text
用 prc-patent-law-system 解释第 71 条的赔偿阶梯和惩罚性赔偿
prc-patent-law-system：优先权的期限、副本、恢复分别是法、细则、指南怎么规定的
PCT 申请进入中国的期限、进入日怎么定、逾期后果，顺带说一下审查员核对进入声明看哪些项
耽误了指定期限还能恢复吗？两条路径的窗口分别是多久（细则第 6 条 + 指南第五部分第七章）
讲一下创造性三步法，并举一个区别特征是公知常识的例子（gl-p2-ch04）
无效程序里专利权人能怎样修改权利要求？法、细则、指南各怎么说
细则第 57 条的主动修改窗口对发明和实用新型分别是多久
reg-ch04 和 gl-p4-ch03 对无效请求补充证据的 1 个月期限分别怎么写
```

问法可以用主题、法条号（第 N 条）、细则条号、指南节号（第二部分第四章第 3.2.1.1 节），或章节 ID（`law-ch02` / `reg-ch04` / `gl-p2-ch04`）。主题查不到时先看 `index.md` 的合一主题索引；问条文时用 `crosswalk.md` 定位章节。

## 目录结构

```text
.
├── README.md
├── 专利法2020.pdf                       # 来源文本
├── 专利审查指南2023.pdf                 # 来源文本
├── 专利法实施细则2023.txt               # 来源文本（国家知识产权局网站正文，去除页面导航）
└── skills/
    └── prc-patent-law-system/
        ├── SKILL.md                     # 入口：路由表、三层核心规则（按生命周期）、章节索引、三层并列主题索引
        ├── index.md                     # 59 章完整索引（三层各一表）+ 三层合一主题索引 + 提取质量说明
        ├── crosswalk.md                 # 条文对照：法第 N 条 / 细则第 N 条 → 所在章节与引用它的另两层章节
        ├── glossary.md                  # 术语表 540 条，拼音排序，标注【法】【细则】【指南】
        ├── patterns.md                  # 43 个程序模式，按程序阶段分十组
        ├── cheatsheet.md                # 三层并列的期限与后果表、三类专利对比、数字门槛、决策规则、决策树、失权信号
        └── chapters/
            ├── law-ch01-总则.md … law-ch08-附则.md                 # 专利法 8 章
            ├── reg-ch01-总则.md … reg-ch13-附则.md                 # 实施细则 13 章
            └── gl-p1-ch01-… … gl-p6-ch02-…                        # 审查指南 38 章（部分-章-标题）
```

每个章节文件结构相同：来源层与章节 ID → 核心要点 → 引入的框架（每条规则带节号和法/细则条款）→ 关键概念 → 思维模型 → 反模式 → 实例演练 → 关键要点 → 关联（另两层的对应章节）。规则后面的引注写法：`法22.3` = 专利法第 22 条第 3 款；`细则57.3` = 实施细则第 57 条第 3 款（2023 年修订后的编号）；`第3.2.1.1节` = 该章内的节号。

## 生成方法

1. 用 book-to-skill 的 `scripts/extract.py` 提取文本：两个 PDF 用 pdf-inspector 引擎（原生文本，无需 OCR），实施细则用其 HTML 解析路径（bs4）读取国家知识产权局网页正文；按目录、页眉或章标题切成章节片段。
2. 每章按 book-to-skill 的章节模板独立蒸馏（指南的 38 章由并行子任务生成，每章一个），只写规则、标准、期限、后果，不复制原文，每条规则回溯到节号与法条。
3. 先分别生成三个单文本技能，再按 book-to-skill 的 Update / Fold-in 合并规则合成一个技能：章节文件原样保留、加层前缀（`law-` / `reg-` / `gl-`）并把所有跨技能引用改写为技能内章节 ID；术语表合并并按拼音重排、标注来源层；模式按程序阶段重新分组；主题索引合并；由章节文件中的引注自动汇总出条文对照表 `crosswalk.md`；`SKILL.md` 和 `cheatsheet.md` 重写为三层并列的形式，`SKILL.md` 控制在 4K tokens 以内。
4. 每次提交前跑 book-to-skill 自带的 `tools/scan_generated_skill.py`（注入/越权模式扫描）和 `tools/validate_skill.py`（frontmatter 与结构校验），并检查全部相对链接与章节 ID 可解析。

## 已知限制

- **内容是结构化摘要，不是原文。** 引用法条或指南节号时请以国家知识产权局发布的官方文本为准；本技能不构成法律意见。
- **PDF 提取的损失。** 化学结构式、马库什通式在提取中损毁，相关例子只有文字描述；外观设计章节没有图；若干整页被提取成错位的表格单元，已按目录重组，具体页码见 `index.md` 末尾的"提取质量说明"。
- **只覆盖三部文本本身。** 不含司法解释、法院判例和地方规定；细则条号用 2023 年修订后的编号；专利法第四次修正决定与实施细则 2023 年修订决定自身的施行日期未载于源文本，技能文件未补写。
- **忠于原文，不补写。** 原文没有规定的内容（例如指南第三部分未载明细则 120 宽限期的长度、第六部分未载明答复驳回通知的期限），文件里明确写"本章未规定"并指向应查的地方，而不是从其他来源补入。
- **条文对照表是引注汇总。** `crosswalk.md` 的"引用"关系来自章节文件里写了的引注，不是法规之间的全部对应关系，可能有漏。

## 更新

法规修订后，用 book-to-skill 的 Update / Fold-in 模式：把新版文本交给 `/book-to-skill`，指向 `skills/prc-patent-law-system`，它会合并新增或修订的章节并重建索引和术语表；新章节沿用对应层的前缀。改动后请重新跑 `tools/scan_generated_skill.py` 和 `tools/validate_skill.py`，并重新生成 `crosswalk.md`。

## 来源与版权

《中华人民共和国专利法》《中华人民共和国专利法实施细则》《专利审查指南》是法律、行政法规和国家机关的规范性文件，依《著作权法》第五条不受著作权保护；本仓库中的两个 PDF 和一个文本文件是其公开发布文本。技能文件是由这些文本生成的结构化摘要，遵循 book-to-skill 的"不复制原文"规则。
