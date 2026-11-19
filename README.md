# Wish to Code · 许愿 → 代码--傻瓜式编程

> 把「我想要个能自动记账的」这种许愿，问清楚、写成规格，再变成能跑起来的代码。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
![Python](https://img.shields.io/badge/Python-3.12%2B-blue.svg)

大多数人用 AI 写代码，卡住的不是写，而是**说不清**。

「帮我做个记账的」——记什么账？手动输入还是导入账单？一个月多少条？出错要停还是要跳过？这些不问，AI 只能猜；猜错了，代码写得再漂亮也是废的。

这个技能就是补这一步：**先替你把话说清楚，再写代码。**

## 它做什么

```
听懂愿望 → 判断规模 → 追问澄清 → 写规格 → 分层实现 → 真跑一遍验证
```

| 步骤 | 做的事 |
| --- | --- |
| 0 判规模 | 脚本级 / 小工具级 / 系统级，默认往小了判，防止为一句话的需求搭一套后端 |
| 1 澄清 | 按 7 类问题追问，一轮最多 3 个，每个都给你选项 |
| 2 归一化 | 需求文档（docx / xlsx / pdf）先转 Markdown |
| 3 任务说明 | 7 段式规格：目标、输入输出、数据结构、模块、步骤、边界、技术栈 |
| 4 结构化 | 实体 / 接口 / 业务逻辑 / 非功能，四张小表 |
| 5 实现 | 按规模选模板，不是一律上框架 |
| 6 验证 | 必须实际执行并贴出输出，禁止只说「已修复」 |
| 7 交付 | 写清我按什么理解的、哪些是假设、要改该动哪 |

## 它不是什么

- 不是代码生成器：不会跳过澄清直接堆代码
- 不是项目脚手架：**不会**为 30 行能解决的事建 `src/` + 数据库 + Docker
- 不是万能的：需求里的业务规则、金额口径、隐私策略，一律问你，不替你编

## 快速开始

### 方式一：装成 Agent Skill

把整个目录放到你的 agent 技能目录：

```bash
# Claude Code / 兼容 Agent Skills 的工具
git clone <this-repo> ~/.claude/skills/wish-to-code
# 或直接复制
cp -r wish-to-code ~/.claude/skills/
```

技能入口是 `SKILL.md`，触发词：模糊需求、自然语言转代码、需求文档转代码、CRUD、写个脚本。

### 方式二：当提示词用

不装也行。把 `SKILL.md` 的内容贴给任何模型当系统提示词，效果一样。

### 然后

直接说你的愿望：

> 我想要个每天自动汇总某个网页价格的小工具。

好的回复长这样：先判断这是「小工具级」，然后问你 3 个问题（价格在页面哪个位置？多久跑一次？价格变了要不要通知你？），确认后给你 7 段规格，再给代码和真实运行输出。

## 目录结构

```
wish-to-code/
├── SKILL.md                        # 技能主文件：8 步工作流与硬性约束
├── references/
│   ├── clarify-checklist.md        # 许愿澄清清单（7 类问题 + 提问原则）
│   ├── task-spec-template.md       # 7 段式编程任务说明书
│   ├── project_template.md         # 系统级：FastAPI + SQLAlchemy 2.0 + Pydantic v2
│   └── script_template.md          # 脚本级 / 小工具级结构与反过度设计清单
├── scripts/
│   ├── markitdown_convert.py       # 文档 → Markdown（需 markitdown）
│   ├── extract_all.py              # 纯标准库抽 docx 正文与表格
│   ├── extract_xlsx.py             # openpyxl 逐 sheet 导出
│   └── analyze_logic.py            # 粗提「如果…则…」、公式、状态迁移
├── examples/
│   └── photo-rename/               # 端到端示例：脚本级 + 幂等 + 实跑验证
├── LICENSE
└── README.md
```

## 内置的两套模板

**系统级**（多人用、要存数据）：`api → services → repositories` 分层，统一 `Result[T]` 返回，错误码 0 / 4xxxx / 5xxxx，SQLAlchemy 2.0 `Mapped[]`，Pydantic v2 请求响应分离，配置走 `pydantic-settings`，pytest 内存库。

**脚本级**（一次性、单文件）：标准库优先，`argparse` + `logging`，默认 UTF-8，输入校验 + 可读报错 + 明确 exit code，重复跑不产生重复数据。

## 示例

见 [`examples/photo-rename/`](examples/photo-rename/)。一个「按修改时间重命名图片」的脚本，演示了三件事：

- 判成脚本级，单文件，没建项目结构
- 先 `--dry-run` 看计划，再真跑
- 重跑幂等：已符合命名的文件自动跳过

## 设计原则

1. **默认往小了判。** 判错了用户会纠正，过度设计是改不回来的债。
2. **能问就不猜。** 尤其是资金、隐私、对外接口、爬取频率。
3. **一轮三问。** 问题多了用户直接放弃，所以要挑「不问就没法动手」的。
4. **依赖要付理由。** 引入任何第三方库前，先回答「标准库为什么不行」。
5. **跑过才算完。** 静态判断不算验证，交付必须带真实命令和输出。

## 依赖

核心零依赖（纯 Markdown + 标准库脚本）。

可选：

```bash
pip install markitdown   # 文档统一转 Markdown
pip install openpyxl     # xlsx 逐表导出
```

## 常见问题

**Q：我的需求已经写得很清楚了，还要走澄清？**
A：会快速过一遍，只问真正缺的。你写得清楚，它就问得少。

**Q：能不能只出规格不写代码？**
A：能。做完第 3 步会停，问你要不要继续。

**Q：支持 Python 以外的语言吗？**
A：模板是 Python 的，方法论与语言无关，换成别的语言只需替换 `project_template.md` 的落地规范。

**Q：需求很大（几十页文档）怎么办？**
A：先出「实体清单 + 接口清单」让你确认，再分批生成代码。

## 作者

**31**

MIT License，详见 [LICENSE](LICENSE)。
