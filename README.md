# Patent-book-to-skill

把三部中国专利法规文本蒸馏成 Agent Skills（[Agent Skills 标准](https://github.com/agentskills/agentskills)），供 Claude Code、GitHub Copilot CLI、Codex、Amp、Hermes、OpenClaw 等支持该标准的编码/办公 agent 按需加载。生成方法是 [book-to-skill](https://github.com/virgiliojr94/book-to-skill)：只提取结构（规则、判断标准、期限、后果、术语），不复制原文。

| 技能 | 来源 | 规模 | 回答什么问题 |
|---|---|---|---|
| [`prc-patent-law-2020`](skills/prc-patent-law-2020/SKILL.md) | 《中华人民共和国专利法》2020 年第四次修正（`专利法2020.pdf`，24 页，82 条） | 8 章，约 1.8 万 tokens | 法条本身：客体、授权条件、申请与审查、期限与无效、许可、保护与赔偿——"第 N 条怎么规定" |
| [`prc-patent-implementing-regulations-2023`](skills/prc-patent-implementing-regulations-2023/SKILL.md) | 《中华人民共和国专利法实施细则》2023 年修订（`专利法实施细则2023.txt`，自国家知识产权局网站抓取，13 章 149 条） | 13 章，约 3.7 万 tokens | 程序上怎么做、期限多长、逾期后果是什么：手续与送达、期限计算与恢复、申请文件格式、优先权与保藏、受理与申请日、初审/实审/修改/驳回/登记、复审与无效、期限补偿、开放/强制许可、奖励报酬、假冒专利与行政处理、登记公报、费用、PCT 与海牙国际申请 |
| [`prc-patent-examination-guidelines-2023`](skills/prc-patent-examination-guidelines-2023/SKILL.md) | 《专利审查指南》2023 年版（`专利审查指南2023.pdf`，613 页，6 部分 38 章） | 38 章，约 16 万 tokens | 审查员按什么标准判断：初审、实审（三性、说明书与权利要求、检索、程序、计算机程序/化学/中药）、复审与无效、PCT 国家阶段、海牙外观设计、事务处理（受理、费用、期限、送达、保密审查、公报、授权与终止、评价报告、开放许可） |

三个技能是同一体系的三层：法条（专利法）→ 程序与期限（实施细则）→ 审查员的判断标准（审查指南）。细则技能和指南技能每章的"关联"一节都映射到另两个技能的章节。

## 安装

技能都在 `skills/<技能名>/` 下，每个目录以 `SKILL.md` 为入口。本仓库是私有仓库，安装时需要能访问它的 GitHub 凭据。

**方式一：skills CLI**（会扫描仓库里的 `SKILL.md`，按名字选择要装的技能）

```bash
npx skills add https://github.com/Nick-JuRan/Patent-book-to-skill --skill prc-patent-law-2020
npx skills add https://github.com/Nick-JuRan/Patent-book-to-skill --skill prc-patent-implementing-regulations-2023
npx skills add https://github.com/Nick-JuRan/Patent-book-to-skill --skill prc-patent-examination-guidelines-2023
```

**方式二：git clone 后链接到 agent 的技能目录**

```bash
git clone https://github.com/Nick-JuRan/Patent-book-to-skill.git
cd Patent-book-to-skill

# 跨 agent 的个人技能目录（Copilot CLI、Amp、Codex 原生发现）
mkdir -p ~/.agents/skills
ln -s "$PWD/skills/prc-patent-law-2020" ~/.agents/skills/prc-patent-law-2020
ln -s "$PWD/skills/prc-patent-implementing-regulations-2023" ~/.agents/skills/prc-patent-implementing-regulations-2023
ln -s "$PWD/skills/prc-patent-examination-guidelines-2023" ~/.agents/skills/prc-patent-examination-guidelines-2023

# Claude Code 只扫描 ~/.claude/skills，再链一次
mkdir -p ~/.claude/skills
ln -s "$PWD/skills/prc-patent-law-2020" ~/.claude/skills/prc-patent-law-2020
ln -s "$PWD/skills/prc-patent-implementing-regulations-2023" ~/.claude/skills/prc-patent-implementing-regulations-2023
ln -s "$PWD/skills/prc-patent-examination-guidelines-2023" ~/.claude/skills/prc-patent-examination-guidelines-2023
```

项目级安装：把 `skills/<技能名>` 复制或链接到项目的 `.claude/skills/`、`.agents/skills/` 或 `.github/skills/`。Hermes 用 `$HERMES_HOME/skills/<分类>/<技能名>`，OpenClaw 用 `${OPENCLAW_STATE_DIR:-~/.openclaw}/skills/<技能名>`。装好后重启 agent 会话（Copilot CLI 用 `/skills reload`）。

## 用法

技能是按需加载的：`SKILL.md` 常驻（核心规则 + 索引），章节文件和术语表、模式、速查表只在需要时读取。

```text
用 prc-patent-law-2020 解释第 71 条的赔偿阶梯和惩罚性赔偿
prc-patent-law-2020：发明、实用新型、外观设计的保护期和优先权期限分别是多久
用 prc-patent-implementing-regulations-2023 说一下 PCT 申请进入中国的期限、进入日怎么定、逾期后果
细则技能：耽误了指定期限还能恢复吗？两条路径的窗口分别是多久
用 prc-patent-examination-guidelines-2023 讲一下创造性三步法，并举一个区别特征是公知常识的例子
指南技能：无效程序里专利权人能怎样修改权利要求？
prc-patent-examination-guidelines-2023 第二部分第三章第 3.2.4 节数值范围的新颖性怎么判
PCT 申请进入中国国家阶段的期限和进入声明要核对哪些项？
```

指南技能的问法也可以用章节号（`p2-ch04` = 第二部分第四章）或节号；主题查不到时先看 `index.md` 的主题索引。

## 目录结构

```text
.
├── README.md
├── 专利法2020.pdf                       # 来源文本
├── 专利审查指南2023.pdf                 # 来源文本
├── 专利法实施细则2023.txt               # 来源文本（国家知识产权局网站正文，去除页面导航）
└── skills/
    ├── prc-patent-law-2020/
    │   ├── SKILL.md                     # 入口：核心规则、章节索引、主题索引
    │   ├── chapters/ch01-总则.md … ch08-附则.md
    │   ├── glossary.md                  # 术语表（约 50 条，按拼音排序，附条文号）
    │   ├── patterns.md                  # 13 个法定程序 / 策略模式
    │   └── cheatsheet.md                # 三类专利对比、期限表、赔偿数字、侵权判断决策树
    ├── prc-patent-implementing-regulations-2023/
    │   ├── SKILL.md                     # 入口：13 章核心规则、带链接的章节索引、主题索引
    │   ├── chapters/ch01-总则.md … ch13-附则.md
    │   ├── glossary.md                  # 术语表（约 100 条，按拼音排序，附章节号与条号）
    │   ├── patterns.md                  # 14 个程序模式
    │   └── cheatsheet.md                # 期限与后果速查、决策规则、数字门槛、失权信号
    └── prc-patent-examination-guidelines-2023/
        ├── SKILL.md                     # 入口：六个部分的核心规则、带链接的章节索引、常用主题入口（< 4K tokens）
        ├── index.md                     # 38 章索引（每章关键规则）+ 六个完整主题索引 + 提取质量说明
        ├── chapters/p1-ch01-… … p6-ch02-…  # 38 章，文件名 = 部分-章-标题
        ├── glossary.md                  # 术语表（约 390 条，按拼音排序，附章节号与节号）
        ├── patterns.md                  # 16 个审查与应对的程序模式
        └── cheatsheet.md                # 期限速查、判断规则、数字门槛、决策树、失权信号
```

每个章节文件结构相同：核心要点 → 引入的框架（每条规则带指南节号和法/细则条款）→ 关键概念 → 思维模型 → 反模式 → 实例演练 → 关键要点 → 关联。规则后面的引注写法：`法22.3` = 专利法第 22 条第 3 款；`细则57.3` = 实施细则第 57 条第 3 款（2023 年修订后的编号）；`第3.2.1.1节` = 该章内的节号。

## 生成方法

1. 用 book-to-skill 的 `scripts/extract.py` 提取文本：两个 PDF 用 pdf-inspector 引擎（原生文本，无需 OCR），实施细则用其 HTML 解析路径（bs4）读取国家知识产权局网页正文；按目录、页眉或章标题切成章节片段。
2. 每章按 book-to-skill 的章节模板独立蒸馏（指南的 38 章由并行子任务生成，每章一个），只写规则、标准、期限、后果，不复制原文，每条规则回溯到节号与法条。
3. 汇总成 `SKILL.md`（核心规则 + 索引）、术语表、程序模式、速查表；`SKILL.md` 控制在 book-to-skill 的 4K tokens 预算内，超出的导航内容放到 `index.md`。
4. 每次提交前跑 book-to-skill 自带的 `tools/scan_generated_skill.py`（注入/越权模式扫描）和 `tools/validate_skill.py`（frontmatter 与结构校验），并检查全部相对链接可解析。

生成过程按 PR 增量合并（专利法一次；指南按第二、四、一、三+六、五部分各一次，再一次收尾整理；实施细则分第一至四章、第五至十三章、收尾三次），每个 PR 的描述里记录了该部分的提取质量问题和"原文没写、文件也没补"的地方。

## 已知限制

- **内容是结构化摘要，不是原文。** 引用法条或指南节号时请以国家知识产权局发布的官方文本为准；这两个技能不构成法律意见。
- **PDF 提取的损失。** 化学结构式、马库什通式在提取中损毁，相关例子只有文字描述；外观设计章节没有图；若干整页被提取成错位的表格单元，已按目录重组，具体页码见指南技能 `index.md` 末尾的"提取质量说明"。
- **只覆盖三部文本本身。** 不含司法解释、法院判例和地方规定；细则条号用 2023 年修订后的编号；实施细则 2023 年修订决定自身的施行日期未载于抓取的正文，技能文件未补写。
- **忠于原文，不补写。** 原文没有规定的内容（例如指南第三部分未载明细则 120 宽限期的长度、第六部分未载明答复驳回通知的期限），文件里明确写"本章未规定"并指向应查的地方，而不是从其他来源补入。

## 更新

法规修订后，用 book-to-skill 的 Update / Fold-in 模式：把新版 PDF 交给 `/book-to-skill`，指向已有技能目录，它会合并新增或修订的章节并重建索引和术语表。改动后请重新跑 `tools/scan_generated_skill.py` 和 `tools/validate_skill.py`。

## 来源与版权

《中华人民共和国专利法》《中华人民共和国专利法实施细则》《专利审查指南》是法律、行政法规和国家机关的规范性文件，依《著作权法》第五条不受著作权保护；本仓库中的两个 PDF 和一个文本文件是其公开发布文本。技能文件是由这些文本生成的结构化摘要，遵循 book-to-skill 的"不复制原文"规则。
