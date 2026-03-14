# Claude Skill 创建指引

> 本指引将帮助你根据自然语言需求描述，创建符合 Claude 官方规范的标准 Skill 技能包。

## 目录

- [Skill 概述](#skill-概述)
- [创建流程](#创建流程)
- [目录结构规范](#目录结构规范)
- [SKILL.md 编写指南](#skillmd-编写指南)
- [资源文件组织](#资源文件组织)
- [最佳实践](#最佳实践)
- [验证与打包](#验证与打包)

---

## Skill 概述

### 什么是 Skill？

Skill 是模块化的能力包，用于扩展 Claude 的功能。每个 Skill 打包了指令、元数据和可选资源（脚本、模板），Claude 会在相关场景下自动使用它们。

### Skill 的价值

- **专业化能力**：将通用 Claude 转化为特定领域的专家
- **复用性**：创建一次，自动应用于相关对话
- **可组合性**：多个 Skills 可以组合实现复杂工作流
- **渐进式加载**：三级加载机制，高效利用上下文窗口

### 三级加载机制

| 级别 | 何时加载 | Token 成本 | 内容 |
|------|---------|-----------|------|
| **Level 1: 元数据** | 启动时（始终加载） | ~100 tokens/Skill | YAML frontmatter 中的 `name` 和 `description` |
| **Level 2: 指令** | Skill 被触发时 | <5k tokens | SKILL.md 主体的工作流程和指导 |
| **Level 3: 资源** | 按需加载 | 实际无限制 | 脚本可以不加载到上下文直接执行 |

---

## 创建流程

### 步骤概览

```
需求理解 → 资源规划 → 初始化目录 → 编写 SKILL.md → 添加资源 → 验证打包 → 迭代改进
```

**重要提示**：按顺序执行这些步骤。只有在有明确理由时才跳过某个步骤。

### 详细步骤

#### 第 1 步：理解需求与具体场景

**目标**：通过具体示例明确 Skill 的使用模式

**关键问题**：
- 这个 Skill 应该支持什么功能？
- 可以给出一些具体的使用示例吗？
- 用户会如何描述才能触发这个 Skill？
- 有哪些边界情况需要考虑？

**💡 交互提示**：如果是 AI 在询问用户，为避免让用户感到过载，不要一次性问太多问题。从最重要的问题开始，根据需要逐步跟进。

**示例**：
```
需求：创建一个图片编辑 Skill

具体场景：
- "去除这张图片的红眼效果"
- "旋转这张图片 90 度"
- "调整图片亮度和对比度"
- "裁剪图片到指定尺寸"
```

**输出**：清晰的功能范围和使用场景列表

---

#### 第 2 步：规划可复用资源

**目标**：分析每个场景，识别需要的可复用资源

**分析维度**：
1. 哪些操作需要重复编写相同代码？ → `scripts/`
2. 哪些信息需要作为参考文档？ → `references/`
3. 哪些文件会用于输出？ → `assets/`

**资源类型决策树**：

```
需要这个资源吗？
│
├─ 会重复编写相同代码？
│  └─ Yes → scripts/ (如 rotate_pdf.py)
│
├─ 需要参考文档信息？
│  └─ Yes → references/ (如 schema.md, api_docs.md)
│
└─ 作为输出模板使用？
   └─ Yes → assets/ (如 template.html, logo.png)
```

**示例**：
```
Skill: pdf-editor

场景分析：
- "旋转这个 PDF" → 每次都要重写旋转代码
  → 需要: scripts/rotate_pdf.py

- "查询 BigQuery 中的用户数据" → 每次都要重新查找表结构
  → 需要: references/schema.md

- "创建一个 Todo 应用" → 每次都要写样板 HTML/React
  → 需要: assets/frontend-template/
```

**输出**：需要创建的资源文件清单

---

#### 第 3 步：初始化 Skill 目录

**⚠️ 重要：优先使用自动化工具**

如果有 `init_skill.py` 脚本可用，**务必使用它来初始化新 Skill**。这个脚本会自动创建规范的目录结构和模板文件。

**使用 init_skill.py（推荐）**：

```bash
# 基本用法
scripts/init_skill.py <skill-name> --path <output-directory>

# 示例
scripts/init_skill.py data-analyzer --path ./skills/
```

脚本会自动：
- ✅ 创建标准目录结构
- ✅ 生成 SKILL.md 模板（含正确的 frontmatter）
- ✅ 创建示例资源目录（scripts/, references/, assets/）
- ✅ 添加示例文件供参考

**初始化后的任务**：
1. 自定义或删除生成的示例文件
2. 根据实际需求编辑 SKILL.md
3. 添加必要的脚本、参考文档和资源

---

**手动创建（无 init_skill.py 时）**：

**标准目录结构**：

```
skill-name/
├── SKILL.md                    # 必需：主要指令文件
├── scripts/                    # 可选：可执行脚本
│   ├── example_script.py
│   └── helper.sh
├── references/                 # 可选：参考文档
│   ├── api_documentation.md
│   ├── schema.md
│   └── examples.md
└── assets/                     # 可选：输出资源
    ├── templates/
    ├── images/
    └── fonts/
```

**命名规范**：
- Skill 名称：小写字母、数字、连字符，最多 64 字符
- 不能包含：`anthropic`、`claude` 等保留词
- 示例：`pdf-editor`, `data-analyzer`, `frontend-builder`

---

#### 第 4 步：编写 SKILL.md

**核心原则**：Skill 是为另一个 Claude 实例使用的。专注于包含对 Claude 有益且非显而易见的信息。考虑哪些过程知识、领域特定细节或可复用资源能帮助另一个 Claude 实例更有效地执行这些任务。

**必需结构**：

```markdown
---
name: skill-name
description: 简要描述此 Skill 的功能以及何时使用它。这是 Claude 判断是否触发此 Skill 的关键。
---

# Skill 名称

## 概述
[1-2 段简要说明此 Skill 的目的]

## 何时使用
[明确列出应该触发此 Skill 的场景]

## 工作流程

### 主要功能 1
[具体步骤说明]

### 主要功能 2
[具体步骤说明]

## 可用资源

### 脚本
- `scripts/xxx.py`: [说明用途和用法]

### 参考文档
- `references/xxx.md`: [说明内容]

### 资源文件
- `assets/xxx/`: [说明用途]

## 示例
[提供 2-3 个具体使用示例]

## 注意事项
[列出重要的限制、边界条件或最佳实践]
```

**编写要点**：

**SKILL.md 主体**：
- ✅ 使用**祈使句/不定式**形式（动词开头）
- ✅ 客观、指令式语言："要完成 X，执行 Y"
- ❌ 避免第二人称："你应该..."
- ❌ 避免口语化表达

**YAML description**：
- ✅ 使用**第三人称**："This skill should be used when..."
- ✅ 包含功能描述和触发条件
- ❌ 避免第二人称："Use this skill when..."

**description 编写指南**：

```markdown
好的 description（第三人称，功能+触发条件）:
✅ "Extract text and tables from PDF files, fill forms, merge documents. This skill should be used when working with PDF files or when the user mentions PDFs, forms, or document extraction."

✅ "Analyze CSV and Excel data files. Calculate statistics, identify trends, generate visualizations. This skill should be used when the user mentions data analysis, uploads data files, or requests statistical calculations."

包含三个关键要素：
1. 功能描述：做什么（动词开头）
2. 触发条件：何时使用（第三人称："This skill should be used when..."）
3. 关键词：帮助识别的触发词

不好的 description:
❌ "PDF tool" - 太简短，缺少触发条件
❌ "Use this skill for PDFs" - 使用了第二人称而非第三人称
❌ "This skill helps with PDFs" - 缺少具体功能和触发条件
```

---

#### 第 5 步：添加资源文件

##### A. Scripts (scripts/)

**何时使用**：
- 相同代码被重复编写
- 需要确定性可靠性
- 复杂的数据处理逻辑

**优势**：
- Token 高效（执行时不加载到上下文）
- 确定性执行
- 可维护性强

**示例：PDF 旋转脚本**

```python
# scripts/rotate_pdf.py
"""
旋转 PDF 文件的实用脚本

使用方法：
    python scripts/rotate_pdf.py <input.pdf> <output.pdf> <angle>

参数：
    input.pdf: 输入 PDF 文件路径
    output.pdf: 输出 PDF 文件路径
    angle: 旋转角度 (90, 180, 270)
"""
import sys
from PyPDF2 import PdfReader, PdfWriter

def rotate_pdf(input_path, output_path, angle):
    reader = PdfReader(input_path)
    writer = PdfWriter()
    
    for page in reader.pages:
        page.rotate(angle)
        writer.add_page(page)
    
    with open(output_path, 'wb') as output_file:
        writer.write(output_file)

if __name__ == "__main__":
    if len(sys.argv) != 4:
        print(__doc__)
        sys.exit(1)
    
    rotate_pdf(sys.argv[1], sys.argv[2], int(sys.argv[3]))
```

**SKILL.md 中的引用**：

```markdown
## 可用脚本

### PDF 旋转
使用 `scripts/rotate_pdf.py` 旋转 PDF 文件：

\```bash
python scripts/rotate_pdf.py input.pdf output.pdf 90
\```

参数：
- angle: 旋转角度 (90, 180, 270)
```

---

##### B. References (references/)

**何时使用**：
- 数据库架构文档
- API 规格说明
- 公司政策或领域知识
- 详细的工作流程指南

**优势**：
- 保持 SKILL.md 精简
- 按需加载（只在 Claude 判断需要时加载）
- 适合大量文档（>10k 字）

**大文件最佳实践**：
对于超过 10k 字的文件，在 SKILL.md 中提供 grep 搜索模式：

```markdown
## 参考文档

### 数据库架构
完整的数据库架构定义在 `references/database_schema.md` 中。

**快速查找**：
- 查找特定表：`grep "^## Table:" references/database_schema.md`
- 查找关系：`grep "Foreign Key" references/database_schema.md`
```

**示例：API 文档**

```markdown
# references/api_documentation.md

## 认证

所有 API 请求需要在 header 中包含认证令牌：

\```
Authorization: Bearer YOUR_TOKEN
\```

## 端点

### GET /api/users

获取用户列表

**参数**：
- `page` (int): 页码，默认 1
- `limit` (int): 每页数量，默认 20

**响应**：
\```json
{
  "users": [...],
  "total": 100,
  "page": 1
}
\```

### POST /api/users

创建新用户

**请求体**：
\```json
{
  "name": "string",
  "email": "string",
  "role": "string"
}
\```
```

**避免重复**：
❌ 不要在 SKILL.md 和 references 中重复相同信息
✅ 核心流程在 SKILL.md，详细参考在 references

---

##### C. Assets (assets/)

**何时使用**：
- 模板文件
- 品牌资源（logo、图标）
- 样板代码
- 字体文件
- 示例文档

**优势**：
- 不加载到上下文
- 直接用于输出
- 支持二进制文件

**组织方式**：

```
assets/
├── templates/
│   ├── email_template.html
│   └── report_template.docx
├── images/
│   ├── logo.png
│   └── icon.svg
├── frontend-template/
│   ├── index.html
│   ├── style.css
│   ├── app.js
│   └── package.json
└── fonts/
    └── custom-font.ttf
```

**示例：前端模板**

```html
<!-- assets/frontend-template/index.html -->
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>App Template</title>
    <link rel="stylesheet" href="style.css">
</head>
<body>
    <div id="app">
        <!-- 应用内容将在这里渲染 -->
    </div>
    <script src="app.js"></script>
</body>
</html>
```

**SKILL.md 中的引用**：

```markdown
## 前端模板

创建新的前端应用时，复制 `assets/frontend-template/` 目录作为起点：

\```bash
cp -r assets/frontend-template/ ./new-app/
cd new-app
# 根据需求自定义文件
\```

模板包含：
- `index.html`: 主 HTML 文件
- `style.css`: 基础样式
- `app.js`: JavaScript 入口点
- `package.json`: 依赖配置
```

---

#### 第 6 步：验证与打包

**自动验证**：

打包脚本会自动验证以下内容：
- YAML frontmatter 格式和必需字段
- Skill 命名规范
- description 完整性和质量
- 文件组织和资源引用

**手动检查清单**：

```markdown
□ SKILL.md 存在且格式正确
□ name 和 description 符合规范
□ description 清晰说明功能和触发条件
□ 所有引用的资源文件都存在
□ 脚本有执行权限且包含使用说明
□ 没有冗余或未使用的文件
□ 文档使用祈使句/不定式形式
□ 提供了具体使用示例
```

**使用打包脚本**：

```bash
# 基本用法
scripts/package_skill.py <path/to/skill-folder>

# 指定输出目录
scripts/package_skill.py <path/to/skill-folder> ./dist

# 输出：skill-name.zip
```

打包成功后，会生成一个可分发的 `.zip` 文件。

---

## 目录结构规范

### 完整示例

```
data-analyzer/
├── SKILL.md                           # 主指令文件
├── scripts/                           # 可执行脚本
│   ├── analyze_csv.py                 # CSV 分析脚本
│   ├── generate_report.py             # 报告生成脚本
│   └── utils.py                       # 工具函数
├── references/                        # 参考文档
│   ├── data_schema.md                 # 数据架构说明
│   ├── analysis_methods.md            # 分析方法文档
│   └── examples/                      # 示例集合
│       ├── example1.md
│       └── example2.md
└── assets/                            # 输出资源
    ├── templates/
    │   ├── report_template.html       # HTML 报告模板
    │   └── chart_config.json          # 图表配置
    └── styles/
        └── report.css                 # 报告样式
```

### 必需文件

- ✅ `SKILL.md` - 唯一必需文件

### 可选目录

- `scripts/` - Python、Bash 等可执行脚本
- `references/` - Markdown 参考文档
- `assets/` - 模板、图片等资源文件

---

## SKILL.md 编写指南

### 元数据规范 (YAML Frontmatter)

```yaml
---
name: your-skill-name              # 必需
description: 功能描述和触发条件     # 必需
version: 1.0.0                     # 可选：版本号
author: Your Name                  # 可选：作者
tags: [tag1, tag2]                 # 可选：标签
---
```

**name 要求**：
- 最多 64 字符
- 只能包含小写字母、数字、连字符
- 不能包含 XML 标签
- 不能包含保留词：`anthropic`, `claude`

**description 要求**：
- 非空
- 最多 1024 字符
- 不能包含 XML 标签
- 应包含功能描述和触发条件
- **使用第三人称描述**（例如："This skill should be used when..." 而不是 "Use this skill when..."）

### 内容结构建议

```markdown
# Skill 标题

## 概述
[简要说明目的，1-2 段]

## 何时使用
明确列出触发场景：
- 当用户需要...
- 当提到...关键词
- 处理...类型的任务

## 核心能力
### 能力 1: [名称]
[详细说明和步骤]

### 能力 2: [名称]
[详细说明和步骤]

## 工作流程
提供标准操作流程：
1. 第一步：...
2. 第二步：...
3. 第三步：...

## 可用资源

### 脚本
- `scripts/xxx.py`: 用途和用法
- `scripts/yyy.sh`: 用途和用法

### 参考文档
- `references/aaa.md`: 内容说明
- `references/bbb.md`: 内容说明

### 资源文件
- `assets/templates/`: 模板集合
- `assets/xxx.png`: 资源说明

## 使用示例

### 示例 1: [场景名称]
\```
用户输入："..."
预期输出：...
使用的资源：...
\```

### 示例 2: [场景名称]
\```
用户输入："..."
预期输出：...
\```

## 限制和注意事项
- 注意事项 1
- 限制条件 2
- 最佳实践 3

## 技术要求
- Python 3.11+
- 必需的包：pandas, numpy
- 运行环境：无网络访问
```

### 写作风格

**✅ 推荐**：
```markdown
## 工作流程
执行以下步骤处理 PDF：
1. 使用 pdfplumber 打开文件
2. 提取每页的文本内容
3. 应用 OCR 处理扫描页面
4. 保存结果到 JSON 格式

调用脚本时，传递文件路径作为第一个参数。
```

**❌ 避免**：
```markdown
## 工作流程
你应该这样处理 PDF：
1. 你需要用 pdfplumber 打开文件
2. 你要提取文本...

如果你想调用脚本，你可以这样做...
```

---

## 资源文件组织

### Scripts 最佳实践

**结构**：
```python
#!/usr/bin/env python3
"""
脚本用途的简短描述

使用方法：
    python scripts/script_name.py <arg1> <arg2>

参数：
    arg1: 参数说明
    arg2: 参数说明

示例：
    python scripts/analyze.py data.csv output.json
"""

import sys
import argparse

def main():
    parser = argparse.ArgumentParser(description='脚本描述')
    parser.add_argument('input', help='输入文件')
    parser.add_argument('output', help='输出文件')
    args = parser.parse_args()
    
    # 主逻辑
    process(args.input, args.output)

if __name__ == "__main__":
    main()
```

**要点**：
- 包含清晰的 docstring
- 提供使用示例
- 参数验证和错误处理
- 返回有意义的错误消息

### References 最佳实践

**组织方式**：
```
references/
├── README.md                  # 索引文件
├── getting_started.md         # 快速开始
├── api/                       # API 文档
│   ├── authentication.md
│   ├── endpoints.md
│   └── errors.md
├── schemas/                   # 数据架构
│   ├── user_schema.md
│   └── order_schema.md
└── examples/                  # 示例集合
    ├── basic_usage.md
    └── advanced_usage.md
```

**索引文件示例**：
```markdown
# 参考文档索引

## API 文档
- [认证](api/authentication.md) - API 认证方法
- [端点](api/endpoints.md) - 所有可用端点
- [错误处理](api/errors.md) - 错误代码说明

## 数据架构
- [用户架构](schemas/user_schema.md) - 用户数据模型
- [订单架构](schemas/order_schema.md) - 订单数据模型

## 使用示例
- [基础用法](examples/basic_usage.md)
- [高级用法](examples/advanced_usage.md)
```

### Assets 最佳实践

**命名规范**：
- 使用描述性名称
- 小写字母和连字符
- 包含版本号（如适用）

```
assets/
├── templates/
│   ├── email-welcome-v1.html
│   ├── email-notification-v1.html
│   └── report-monthly-v2.html
├── images/
│   ├── logo-primary.png
│   ├── logo-secondary.png
│   └── icon-16x16.png
└── boilerplate/
    └── react-app/
        ├── public/
        ├── src/
        └── package.json
```

---

## 最佳实践

### 1. 保持 SKILL.md 精简

**问题**：SKILL.md 过大会消耗过多 tokens

**解决方案**：
- SKILL.md：核心工作流程和指导（<5k tokens）
- references/：详细文档和参考资料
- scripts/：可执行逻辑

**示例**：
```markdown
## 数据库查询

查询用户数据时，参考 `references/database_schema.md` 了解完整的表结构。

常用查询模式：
1. 按 ID 查询：使用 `users` 表的主键
2. 按邮箱查询：使用 `email` 索引字段

完整的架构文档包含所有表、字段、关系和索引信息。
```

### 2. 渐进式披露信息

**原则**：只在需要时加载详细信息

**实现**：
```markdown
## 处理 PDF 表单

对于简单 PDF 提取，使用 pdfplumber 库（预装）。

对于高级表单填充功能，参考 `references/forms.md` 获取详细指南。

对于复杂的表单验证，使用 `scripts/validate_form.py`。
```

### 3. 提供具体示例

**不够具体**：
```markdown
此 Skill 用于数据分析。
```

**足够具体**：
```markdown
## 使用示例

### 示例 1：分析销售数据
用户："分析这个 CSV 文件的销售趋势"

工作流程：
1. 使用 `scripts/analyze_csv.py sales.csv` 加载数据
2. 识别时间序列列（日期/时间戳）
3. 计算月度/季度趋势
4. 生成可视化图表
5. 输出洞察报告

### 示例 2：比较多个数据集
用户："比较今年和去年的销售数据"

工作流程：
1. 加载两个数据集
2. 标准化时间范围
3. 计算同比增长率
4. 生成对比图表
```

### 4. 明确触发条件

**description 应该明确**：
```yaml
description: Analyze CSV, Excel, and JSON data files. Calculate statistics, identify trends, generate visualizations. This skill should be used when the user mentions data analysis, asks for insights, uploads data files, or requests statistical calculations.
```

**包含的关键词**：
- 功能：analyze, calculate, visualize
- 文件类型：CSV, Excel, JSON
- 触发词：data analysis, insights, statistics

### 5. 处理运行环境限制

**限制**：
- ❌ 无网络访问
- ❌ 无法安装运行时包
- ✅ 只能使用预装包

**预装包列表**（参考）：
- 数据科学：pandas, numpy, scipy, scikit-learn
- 可视化：matplotlib, seaborn
- 文件处理：openpyxl, pillow, pypdf
- 工具：tqdm, python-dateutil

**在 SKILL.md 中说明**：
```markdown
## 技术要求

### 可用的 Python 包
此 Skill 使用以下预装包：
- pandas: 数据处理
- matplotlib: 可视化
- numpy: 数值计算

### 限制
- 无网络访问：不能调用外部 API
- 无法安装新包：只能使用预装包
- 5GiB 磁盘空间
- 5GiB RAM
```

### 6. 脚本独立性

**原则**：每个脚本应该可以独立运行

**好的脚本**：
```python
#!/usr/bin/env python3
"""独立的 PDF 旋转脚本"""
import sys
from PyPDF2 import PdfReader, PdfWriter

def rotate_pdf(input_path, output_path, angle):
    """旋转 PDF 文件"""
    # 完整的实现
    pass

if __name__ == "__main__":
    # 可以直接运行
    if len(sys.argv) != 4:
        print("Usage: python rotate_pdf.py input.pdf output.pdf angle")
        sys.exit(1)
    rotate_pdf(sys.argv[1], sys.argv[2], int(sys.argv[3]))
```

### 7. 避免信息重复

**问题**：SKILL.md 和 references/ 中的重复内容

**解决方案**：
- SKILL.md：简要说明 + 引用链接
- references/：完整详细信息

**示例**：
```markdown
<!-- SKILL.md -->
## API 端点

主要端点：
- `/users` - 用户管理
- `/orders` - 订单管理

详细的端点文档、参数和响应格式见 `references/api_endpoints.md`。
```

---

## 验证与打包

### 自动验证检查项

打包脚本 `package_skill.py` 会验证：

1. **结构验证**
   - SKILL.md 文件存在
   - YAML frontmatter 格式正确
   - 必需字段存在（name, description）

2. **命名验证**
   - name 符合命名规范
   - 长度不超过 64 字符
   - 无保留词

3. **描述验证**
   - description 非空
   - 长度不超过 1024 字符
   - 无 XML 标签

4. **文件组织**
   - 目录结构合理
   - 资源文件引用正确

### 验证错误示例

**错误 1：缺少必需字段**
```
Error: Missing required field 'description' in YAML frontmatter
Fix: Add description field to SKILL.md frontmatter
```

**错误 2：名称不符合规范**
```
Error: Skill name 'MySkill' contains uppercase letters
Fix: Change name to 'my-skill' (lowercase with hyphens)
```

**错误 3：description 过长**
```
Error: Description exceeds 1024 characters (current: 1250)
Fix: Shorten description to be more concise
```

### 手动测试

**创建测试场景**：

```bash
# 1. 解压 Skill 包到测试目录
unzip skill-name.zip -d ~/.claude/skills/

# 2. 创建测试对话
# 使用 Claude 并提供触发该 Skill 的请求

# 3. 验证行为
# - Skill 是否被正确触发？
# - 脚本是否正常执行？
# - 输出是否符合预期？
```

**测试检查清单**：
```markdown
□ Skill 在相关场景下被自动触发
□ 不在无关场景下误触发
□ 所有脚本能正常执行
□ 参考文档能被正确加载
□ 资源文件能被正确使用
□ 错误处理恰当
□ 输出质量符合预期
```

---

## 完整示例

### 示例 1：数据分析 Skill

**需求**：创建一个能分析 CSV/Excel 数据的 Skill

**第 1 步：理解场景**
```
场景 1："分析这个 CSV 文件的销售趋势"
场景 2："计算这个 Excel 表格的统计摘要"
场景 3："可视化数据中的关键指标"
```

**第 2 步：规划资源**
```
需要的资源：
- scripts/analyze_data.py - 执行数据分析
- scripts/generate_charts.py - 生成可视化
- references/analysis_methods.md - 分析方法说明
- assets/templates/report.html - 报告模板
```

**第 3 步：目录结构**
```
data-analyzer/
├── SKILL.md
├── scripts/
│   ├── analyze_data.py
│   └── generate_charts.py
├── references/
│   └── analysis_methods.md
└── assets/
    └── templates/
        └── report.html
```

**第 4 步：SKILL.md**
```markdown
---
name: data-analyzer
description: Analyze CSV and Excel data files. Calculate statistics, identify trends, generate visualizations and reports. This skill should be used when the user mentions data analysis, statistics, data insights, or uploads CSV/Excel files.
---

# 数据分析 Skill

## 概述
此 Skill 专门用于分析结构化数据文件（CSV、Excel），提供统计分析、趋势识别和可视化功能。

## 何时使用
- 用户上传 CSV 或 Excel 文件
- 用户要求数据分析或统计摘要
- 用户提到"趋势"、"洞察"、"可视化"等关键词
- 需要生成数据报告

## 核心能力

### 1. 数据加载和清洗
- 支持 CSV、Excel (xlsx/xls) 格式
- 自动检测列类型
- 处理缺失值

### 2. 统计分析
使用 `scripts/analyze_data.py` 计算：
- 描述性统计（均值、中位数、标准差）
- 分布分析
- 相关性分析
- 异常值检测

### 3. 可视化
使用 `scripts/generate_charts.py` 生成：
- 折线图（趋势分析）
- 柱状图（类别对比）
- 散点图（相关性）
- 箱线图（分布）

## 工作流程

1. **加载数据**
   \```bash
   python scripts/analyze_data.py <data_file.csv>
   \```

2. **执行分析**
   - 自动检测数值列
   - 计算基础统计信息
   - 识别潜在趋势

3. **生成可视化**
   \```bash
   python scripts/generate_charts.py <data_file.csv> --output charts/
   \```

4. **创建报告**
   - 使用 `assets/templates/report.html` 模板
   - 插入分析结果和图表
   - 输出 HTML 报告

## 可用资源

### 脚本
- `scripts/analyze_data.py`: 主数据分析脚本
  - 输入：CSV/Excel 文件路径
  - 输出：JSON 格式的统计结果

- `scripts/generate_charts.py`: 图表生成脚本
  - 输入：数据文件和配置
  - 输出：PNG/SVG 图表文件

### 参考文档
- `references/analysis_methods.md`: 详细的分析方法说明

### 资源文件
- `assets/templates/report.html`: HTML 报告模板

## 使用示例

### 示例 1：基础统计分析
用户："分析 sales.csv 并给我统计摘要"

执行步骤：
1. 运行 `python scripts/analyze_data.py sales.csv`
2. 提取关键统计指标
3. 呈现清晰的摘要

### 示例 2：趋势可视化
用户："将这个 Excel 文件的月度趋势可视化"

执行步骤：
1. 识别时间序列列
2. 运行 `python scripts/generate_charts.py data.xlsx --type line`
3. 生成趋势图表

## 技术要求

### 使用的包
- pandas: 数据处理
- numpy: 数值计算
- matplotlib/seaborn: 可视化
- openpyxl: Excel 支持

### 限制
- 最大文件大小：受容器磁盘空间限制（5GB）
- 无网络访问
- 只能使用预装 Python 包

## 注意事项
- 大文件处理可能需要较长时间
- 确保数据文件格式正确（UTF-8 编码）
- 对于复杂分析，参考 `references/analysis_methods.md`
```

**第 5 步：创建脚本**

```python
# scripts/analyze_data.py
#!/usr/bin/env python3
"""
数据分析脚本

使用方法：
    python scripts/analyze_data.py <data_file>

参数：
    data_file: CSV 或 Excel 文件路径

输出：
    JSON 格式的统计结果
"""
import sys
import json
import pandas as pd

def analyze_data(file_path):
    """分析数据文件并返回统计信息"""
    # 根据文件扩展名加载数据
    if file_path.endswith('.csv'):
        df = pd.read_csv(file_path)
    elif file_path.endswith(('.xlsx', '.xls')):
        df = pd.read_excel(file_path)
    else:
        raise ValueError("不支持的文件格式")
    
    # 计算统计信息
    stats = {
        'shape': df.shape,
        'columns': list(df.columns),
        'dtypes': {col: str(dtype) for col, dtype in df.dtypes.items()},
        'missing': df.isnull().sum().to_dict(),
        'numeric_summary': df.describe().to_dict()
    }
    
    return stats

def main():
    if len(sys.argv) != 2:
        print(__doc__)
        sys.exit(1)
    
    file_path = sys.argv[1]
    stats = analyze_data(file_path)
    
    print(json.dumps(stats, indent=2))

if __name__ == "__main__":
    main()
```

**第 6 步：打包**
```bash
scripts/package_skill.py data-analyzer/

# 输出：data-analyzer.zip
```

---

#### 第 7 步：迭代改进

Skill 创建完成后不是终点，而是起点。真实使用后往往会发现改进空间。

**迭代工作流程**：

1. **实际使用**
   - 在真实任务中测试 Skill
   - 观察 Claude 如何使用它
   - 记录表现良好和不足的地方

2. **识别问题**
   - Skill 是否在正确的场景被触发？
   - 是否有误触发的情况？
   - 脚本执行是否顺利？
   - 文档是否足够清晰？
   - 是否有缺失的功能？

3. **确定改进方向**
   - SKILL.md 需要补充哪些说明？
   - 是否需要新增或修改脚本？
   - references 文档是否需要更新？
   - description 是否需要调整以改善触发准确性？

4. **实施改进**
   - 更新相应文件
   - 重新打包
   - 再次测试

5. **持续优化**
   - 收集使用反馈
   - 定期审查和改进
   - 保持 Skill 与需求同步

**改进示例**：

```markdown
问题：Skill 被触发了，但 Claude 没有使用提供的脚本

分析：SKILL.md 中对脚本的说明不够清晰

改进：
- 在 SKILL.md 中增加具体的使用场景说明
- 添加脚本调用示例
- 明确脚本的输入输出格式

---

问题：在不相关的场景下也触发了此 Skill

分析：description 过于宽泛

改进：
- 缩小 description 的范围
- 添加更具体的触发关键词
- 明确排除某些场景
```

**迭代最佳时机**：
- ✅ 使用 Skill 后立即进行（记忆最新鲜）
- ✅ 发现明显问题时
- ✅ 用户需求变化时
- ✅ 定期审查（如每月一次）

---

## 常见问题

### Q1: 如何决定内容应该放在 SKILL.md 还是 references/ 中？

**决策规则**：
- **SKILL.md**：核心工作流程、必要指导、资源索引（<5k tokens）
- **references/**：详细文档、完整规格、大量示例

### Q2: 脚本应该用 Python 还是 Bash？

**建议**：
- **Python**：数据处理、复杂逻辑、文件操作
- **Bash**：简单的文件管理、系统命令组合

### Q3: 如何处理大型参考文档？

**方法 1：分割文件**
```
references/
├── api/
│   ├── authentication.md    (< 5k words)
│   ├── endpoints.md         (< 5k words)
│   └── errors.md            (< 5k words)
```

**方法 2：提供搜索模式**
```markdown
完整的 API 文档在 `references/api_full.md` (50k words)

快速查找：
- `grep "^## Endpoint:" references/api_full.md` - 列出所有端点
- `grep -A 10 "POST /users" references/api_full.md` - 查看特定端点
```

### Q4: Skill 之间可以互相引用吗？

**当前不支持直接引用**，但可以：
- 创建共享的参考文档
- 在 description 中提及相关 Skill
- 使用类似的命名和结构

### Q5: 如何测试 Skill 的触发准确性？

**测试方法**：
1. 编写多样化的测试提示
2. 包括正面（应该触发）和负面（不应触发）案例
3. 观察实际行为
4. 调整 description 改进触发准确性

**测试案例示例**：
```markdown
应该触发的：
✓ "分析这个 CSV 文件"
✓ "给我这些数据的统计摘要"
✓ "可视化销售趋势"

不应触发的：
✗ "写一个数据分析的教程" (创作内容，非分析数据)
✗ "解释什么是数据分析" (概念解释，非执行分析)
```

---

## 附录

### A. 完整的 YAML Frontmatter 字段

```yaml
---
# 必需字段
name: skill-name                    # 小写、连字符、64 字符内
description: 功能和触发条件          # 1024 字符内

# 可选字段（自定义）
version: 1.0.0                      # 版本号
author: Your Name                   # 作者
email: you@example.com              # 联系方式
license: MIT                        # 许可证
tags: [data, analysis, python]      # 标签
created: 2024-01-01                 # 创建日期
updated: 2024-01-15                 # 更新日期
---
```

### B. 预装 Python 包清单

**数据科学**：
- pandas, numpy, scipy, scikit-learn, statsmodels

**可视化**：
- matplotlib, seaborn

**文件处理**：
- pyarrow, openpyxl, xlsxwriter, xlrd, pillow
- python-pptx, python-docx
- pypdf, pdfplumber, pypdfium2, pdf2image, pdfkit, tabula-py
- reportlab[pycairo], Img2pdf

**数学计算**：
- sympy, mpmath

**工具**：
- tqdm, python-dateutil, pytz, joblib

**系统工具**：
- unzip, unrar, 7zip, bc, rg (ripgrep), fd, sqlite

### C. 容器运行环境规格

- **Python 版本**: 3.11.12
- **操作系统**: Linux (x86_64)
- **内存**: 5GiB RAM
- **磁盘**: 5GiB
- **CPU**: 1 CPU
- **网络**: 无网络访问
- **过期时间**: 创建后 30 天

### D. 有用的资源链接

- [Claude Skills 官方文档](https://docs.anthropic.com/en/docs/agents-and-tools/agent-skills)
- [Claude Skills Cookbook](https://github.com/anthropics/claude-cookbooks/tree/main/skills)
- [Code Execution Tool 文档](https://docs.anthropic.com/en/docs/agents-and-tools/tool-use/code-execution-tool)

---

## 总结

创建优秀的 Claude Skill 的关键要素：

1. **明确的触发条件**：description 清晰说明何时使用
2. **精简的 SKILL.md**：核心流程 < 5k tokens
3. **合理的资源组织**：scripts/ references/ assets/ 各司其职
4. **具体的示例**：帮助 Claude 理解使用场景
5. **完整的文档**：脚本用法、参数说明、错误处理
6. **符合规范**：通过自动验证，遵循命名和结构规范

遵循本指引，你就能创建出专业、高效、易维护的 Claude Skills！