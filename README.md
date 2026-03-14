# Skills Hub

🚀 **OpenClaw Skills 集合仓库** - 从 Supabase 迁移的 AI Agent Skills

---

## 📊 统计

| 指标 | 数量 |
|------|------|
| **总 Skills** | 34 |
| ✅ Approved | 29 |
| ⏳ Pending | 2 |
| ❌ Rejected | 1 |
| 🔄 Processing | 1 |

---

## 📁 目录结构

每个 skill 都是一个独立的目录，包含：

```
<skill-name>/
├── metadata.json    # Skill 元数据（名称、描述、分类、状态等）
├── SKILL.md         # Skill 内容（如果没有 zip 文件）
└── ...              # 其他文件（从 zip 解压）
```

---

## 🗂️ 按分类浏览

### ⚡ Productivity (4)

| Skill | 描述 | 状态 |
|-------|------|------|
| [ai partner chat](./ai-partner-chat) | AI Partner Chat是Claude Skills项目，通过双画像和个人笔记，提供个性化、上下文感知的陪伴... | ✅ approved |
| [cloudflare troubleshooting](./cloudflare-troubleshooting) | cloudflare-troubleshooting 是一个通过 API 驱动的证据收集来调查和解决... | ✅ approved |
| [Fetch Web Page Skill](./fetch-web-page-skill) | fetch-web-page-skill 是一个用于从网页获取并提取文本内容的技能，主要功能是帮助用... | ✅ approved |
| [Slack Gif Creator Skill](./slack-gif-creator-skill) | Slack GIF Creator 是一个用于创建符合 Slack 规范的动画 GIF 的工具包，主... | ✅ approved |

### 📦 Picture (1)

| Skill | 描述 | 状态 |
|-------|------|------|
| [Canvas Design Skill](./canvas-design-skill) | canvas-design 是一个用于创建视觉艺术的 Skill，其功能是根据用户输入生成原创的设计... | ✅ approved |

### 🤖 Automation (3)

| Skill | 描述 | 状态 |
|-------|------|------|
| [cli demo generator](./cli-demo-generator) | cli-demo-generator 是一个用于创建动画化 CLI 演示、终端录制或命令行演示 GI... | ✅ approved |
| [Enhanced Skill Creator](./enhanced-skill-creator) | Claude Skill 是模块化能力包，通过三级加载机制（元数据→指令→资源）高效扩展 Claud... | ✅ approved |
| [Skill Creator](./skill-creator) | Skill Creator 是一个指导用户为 Claude 打造「技能包」的元技能。它通过标准化模板... | ✅ approved |

### 💻 Coding (10)

| Skill | 描述 | 状态 |
|-------|------|------|
| [Comprehensive GitHub code review with AI-powered swarm coordination](./comprehensive-github-code-review-with-ai-powered-s) | 该 Skill 是一个基于 AI 群体协调的 GitHub 代码审查工具，通过部署专业化审查代理对 ... | ✅ approved |
| [docx_parser](./docx_parser) | 该 Skill 提供了全面的 .docx 文档创建、编辑和分析功能，支持修订跟踪、评论、格式保留和文... | ❌ rejected |
| [github ops](./github-ops) | 该 Skill 提供了全面的 GitHub 操作功能，主要依赖 `gh` CLI 工具和 GitHu... | ✅ approved |
| [MCP Builder Skill](./mcp-builder-skill) | mcp-builder 是一个用于创建高质量 MCP (Model Context Protocol... | ✅ approved |
| [one-click dev and deploy website](./one-click-dev-and-deploy-website) | Dev & Deploy is an automation-first skill that hel... | ✅ approved |
| [Pair Programming](./pair-programming) | 该Skill提供AI辅助的结对编程功能，支持驾驶员/导航员/切换等多种协作模式，具备实时验证、质量监... | ✅ approved |
| [repomix safe mixer](./repomix-safe-mixer) | repomix-safe-mixer 是一个通过自动检测并移除硬编码凭证来安全打包代码库的 Skil... | ✅ approved |
| [repomix unmixer](./repomix-unmixer) | repomix-unmixer 是一个用于从 repomix 打包的仓库文件中提取文件并恢复原始目录... | ✅ approved |
| [teams channel post writer](./teams-channel-post-writer) | teams-channel-post-writer 是一个用于创建教育性 Teams 频道帖子的 S... | ✅ approved |
| [WebApp Testing Skill](./webapp-testing-skill) | webapp-testing 是一个用于与本地 Web 应用程序交互和测试的工具包，它基于 Play... | ✅ approved |

### ✍️ Writing (5)

| Skill | 描述 | 状态 |
|-------|------|------|
| [deep-reading-analyst-skill](./deep-reading-analyst-skill) | A professional Claude AI skill for deep reading an... | ✅ approved |
| [excalidraw presentation generator](./excalidraw-presentation-generator) | The Excalidraw Presentation Generator Skill automa... | ✅ approved |
| [llm icon finder](./llm-icon-finder) | llm-icon-finder 是一个用于查找和访问 AI/LLM 模型品牌图标的 Skill，主要... | ✅ approved |
| [markdown tools](./markdown-tools) | markdown-tools 是一个将多种文档格式（如 PDF、Word、PowerPoint、Co... | ✅ approved |
| [OpenclawDaily](./openclawdaily) | OpenclawDaily - AI Skill | ⏳ pending |

### 📢 Marketing (2)

| Skill | 描述 | 状态 |
|-------|------|------|
| [Demo Skill](./demo-skill) | 该 demo skill 是一个示例技能，主要用于演示或测试特定功能，其用途包括展示技能的基本操作流... | ✅ approved |
| [dsadas](./dsadas) | dsadas - Processing... | 🔄 processing |

### 📄 Document (3)

| Skill | 描述 | 状态 |
|-------|------|------|
| [Doc Parser Skill](./doc-parser-skill) | 该 Skill 提供了全面的 .docx 文档创建、编辑和分析功能，支持修订跟踪、评论、格式保留和文... | ✅ approved |
| [PDF Parser Skill](./pdf-parser-skill) | 该Skill是一个全面的PDF处理工具包，功能包括提取文本和表格、创建新PDF、合并/拆分文档以及处... | ✅ approved |
| [PPT Parser Skill](./ppt-parser-skill) | pptx Skill 提供了演示文稿的创建、编辑和分析功能，支持处理 .pptx 文件，包括新建演示... | ✅ approved |

### 📊 Data (2)

| Skill | 描述 | 状态 |
|-------|------|------|
| [Excel Parser Skill](./excel-parser-skill) | xlsx 是一个用于创建、编辑和分析电子表格（支持 .xlsx、.xlsm、.csv、.tsv 等格... | ✅ approved |
| [xiaohongshu-search](./xiaohongshu-search) | xiaohongshu-search 通过浏览器自动化实现对小红书笔记的关键词搜索、按时间/热度筛选... | ✅ approved |

### 🔬 Research (2)

| Skill | 描述 | 状态 |
|-------|------|------|
| [Trump & Elon twitter](./trump-elon-twitter) | 该 Skill 的功能是"自动每5分钟抓取特朗普与埃隆·马斯克的 Twitter 消息，基于获取的信... | ⏳ pending |
| [unifuncs](./unifuncs) | Default web reading, AI search, and deep research ... | ✅ approved |

### 🎨 Design (1)

| Skill | 描述 | 状态 |
|-------|------|------|
| [ui desiner](./ui-desiner) | ui desiner - AI Skill | ✅ approved |

---

## 🔍 快速查找

- [ai partner chat](./ai-partner-chat)
- [Canvas Design Skill](./canvas-design-skill)
- [cli demo generator](./cli-demo-generator)
- [cloudflare troubleshooting](./cloudflare-troubleshooting)
- [Comprehensive GitHub code review](./comprehensive-github-code-review-with-ai-powered-s)
- [deep-reading-analyst-skill](./deep-reading-analyst-skill)
- [Demo Skill](./demo-skill)
- [Doc Parser Skill](./doc-parser-skill)
- [docx_parser](./docx_parser)
- [dsadas](./dsadas)
- [Enhanced Skill Creator](./enhanced-skill-creator)
- [excalidraw presentation generator](./excalidraw-presentation-generator)
- [Excel Parser Skill](./excel-parser-skill)
- [Fetch Web Page Skill](./fetch-web-page-skill)
- [github ops](./github-ops)
- [llm icon finder](./llm-icon-finder)
- [markdown tools](./markdown-tools)
- [MCP Builder Skill](./mcp-builder-skill)
- [one-click dev and deploy website](./one-click-dev-and-deploy-website)
- [OpenclawDaily](./openclawdaily)
- [Pair Programming](./pair-programming)
- [PDF Parser Skill](./pdf-parser-skill)
- [PPT Parser Skill](./ppt-parser-skill)
- [repomix safe mixer](./repomix-safe-mixer)
- [repomix unmixer](./repomix-unmixer)
- [Skill Creator](./skill-creator)
- [Slack Gif Creator Skill](./slack-gif-creator-skill)
- [teams channel post writer](./teams-channel-post-writer)
- [Trump & Elon twitter](./trump-elon-twitter)
- [ui desiner](./ui-desiner)
- [unifuncs](./unifuncs)
- [WebApp Testing Skill](./webapp-testing-skill)
- [xiaohongshu-search](./xiaohongshu-search)

---

## 📦 数据来源

- **原始数据库**: Supabase (skills_submissions 表)
- **迁移时间**: 2026-03-14
- **仓库**: https://github.com/vibe-any/skills-hub

---

## 🛠️ 使用方式

每个 skill 目录都包含完整的代码和说明：

1. 浏览上方分类找到感兴趣的 skill
2. 点击进入目录查看详情
3. 阅读 `SKILL.md` 或 `metadata.json` 了解使用方法
4. 根据需要集成到你的 OpenClaw Agent 中

---

## 🤝 贡献

欢迎提交新的 Skills！请通过 Supabase 平台提交你的 Skill。

---

*本仓库由自动化脚本从 Supabase 迁移生成* ✨
