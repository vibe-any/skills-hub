---
name: xiaohongshu-search
description: |
  搜索小红书笔记并提取分享链接。给定关键词后，自动在小红书搜索、按时间/热度筛选、
  逐个打开笔记获取真实分享链接，整理成 Markdown 表格保存到本地。
  当用户说"搜索小红书"、"小红书热帖"、"抓取小红书"、"小红书笔记"时触发。
---

# 小红书搜索与分享链接提取

## 概述

通过浏览器自动化操作小红书 Web 端，完成关键词搜索 → 筛选排序 → 逐条打开笔记 → 提取真实分享链接的完整流程，最终输出结构化的 Markdown 文件。

## 前置条件

执行此 Skill 前，**必须逐项检查**以下依赖，任一不满足则提醒用户先完成安装。

### 必需工具

| 工具 | 用途 | 检查方式 |
|:---|:---|:---|
| **Chrome 浏览器** | 执行所有页面交互 | 确认系统已安装 Chrome（或 Chromium） |
| **Chrome DevTools MCP** | 提供浏览器自动化能力（导航、点击、截图、执行 JS、监听网络请求等） | 确认 MCP 服务已启动并可调用 `navigate_page`、`evaluate_script`、`take_snapshot`、`take_screenshot`、`list_network_requests`、`get_network_request` 等工具 |
| **文件读写工具** | 将结果保存为本地 Markdown 文件 | 确认可调用 `Write`（或 `fs` 等等效文件写入工具） |

### 环境状态检查

在开始操作前执行以下预检：

1. **浏览器可用性**：调用 `list_pages`（或等效命令）确认浏览器连接正常
2. **小红书登录态**：详见核心流程「第零步」——导航到小红书首页，检测登录状态；若未登录则提示用户手动登录并轮询等待（最长 120 秒），登录成功后自动继续
3. **输出目录可写**：检查 `output_path` 的父目录是否存在，不存在则创建

### 可选工具

| 工具 | 用途 | 说明 |
|:---|:---|:---|
| `WebSearch` | 辅助获取笔记内容（当浏览器不可用时的降级方案） | 非核心依赖，仅作兜底 |
| `Shell` | 创建目录等系统操作 | 部分环境可用 `mkdir` 替代 |

> **提示**：如果运行环境不支持 Chrome DevTools MCP（如纯文本 CLI Agent），此 Skill 将无法执行。请提前告知用户安装并配置 [chrome-devtools-mcp](https://github.com/anthropics/chrome-devtools-mcp) 或其他等效的浏览器自动化 MCP Server。

## 输入参数

向用户确认以下参数（括号内为默认值）：

| 参数 | 说明 | 示例 |
|:---|:---|:---|
| `keyword` | 搜索关键词（**必填**） | `openclaw skills` |
| `time_filter` | 时间筛选（`一天内`） | `一天内` / `一周内` / `半年内` / 不限 |
| `sort` | 排序方式（`最多点赞`） | `最多点赞` / `最新` / `综合` |
| `output_path` | 输出文件路径（**必填**） | `./xhs-results.md` |
| `max_count` | 最大采集条数（`20`） | 正整数 |

## 核心流程

### 第零步：登录态检查、Cookie 恢复与等待

在执行任何搜索操作之前，必须先确认用户已登录小红书。本步骤会尝试从本地缓存恢复 Cookie，避免每次都要手动登录。

**Cookie 存储路径约定**：

```
{output_path 所在目录}/.xhs-cookies.json
```

例如输出路径为 `./xhs-results.md`，则 Cookie 文件为 `./.xhs-cookies.json`。

---

**1. 检查是否存在已保存的 Cookie 文件**

读取 Cookie 文件，若存在且非空：

```json
{
  "saved_at": "2026-03-04T17:03:07Z",
  "cookies": "abRequestId=xxx; web_session=xxx; a1=xxx; ..."
}
```

先检查 `saved_at` 距今是否超过 **7 天**。若已过期，跳过恢复，走全新登录流程。

**2. 恢复 Cookie 并验证**

若 Cookie 文件有效，执行以下恢复流程：

```javascript
// 先导航到小红书域名（空白页也可），确保 document.domain 正确
// 然后逐条注入 Cookie
((cookieStr) => {
  const pairs = cookieStr.split('; ');
  for (const pair of pairs) {
    document.cookie = pair + '; domain=.xiaohongshu.com; path=/; max-age=86400';
  }
  return pairs.length + ' cookies set';
})(savedCookieString)
```

> **重要限制**：`document.cookie` 无法设置带 `HttpOnly` 标志的 Cookie（如 `web_session`）。因此 JS 注入只能恢复部分 Cookie。完整恢复方案见下方「推荐：持久化浏览器配置」。

注入后，刷新页面并执行登录态检查（见下方步骤 3）。若检查通过则直接进入第一步；若失败则继续走手动登录流程。

---

**3. 导航到小红书首页，判断登录状态**

导航到：

```
https://www.xiaohongshu.com/explore
```

等待页面加载完成（超时 15 秒），然后取快照判断：

| 登录状态 | 判断依据 |
|:---|:---|
| **已登录** | 快照中存在文本为「我」的链接（`link "我"`），且 `href` 包含 `/user/profile/` |
| **未登录** | 快照中出现「登录」按钮或登录弹窗，或 URL 被重定向到含 `login` 的路径 |

JS 辅助判断：

```javascript
(() => {
  const meLink = document.querySelector('a[href*="/user/profile/"]');
  const loginBtn = document.querySelector('[class*="login"], [class*="Login"]');
  if (meLink) return 'logged_in';
  if (loginBtn) return 'need_login';
  return 'unknown';
})()
```

**4. 若需要登录：提示用户并轮询等待**

向用户发送提示消息：

> 检测到小红书尚未登录。请在浏览器中手动完成登录（支持手机扫码或账号密码登录），登录完成后我将自动继续执行。

进入轮询等待循环：

```
重复（最多 120 秒，每 5 秒检查一次）：
  1. 执行登录检测脚本
  2. 若返回 'logged_in' → 跳出循环
  3. 若超时 → 告知用户「登录等待超时」，终止流程
```

```javascript
(() => {
  return !!document.querySelector('a[href*="/user/profile/"]');
})()
```

**5. 登录成功后：保存 Cookie**

登录成功后立即执行 Cookie 提取与保存：

```
步骤 A：从最近的网络请求中提取完整 Cookie
```

调用 `list_network_requests` 找到任意一个发往 `edith.xiaohongshu.com` 的请求，然后通过 `get_network_request` 读取其请求头中的 `cookie` 字段。该字段包含完整的 Cookie 字符串（含 `HttpOnly` 的 `web_session`）。

```
步骤 B：写入本地文件
```

将提取的 Cookie 连同时间戳保存为 JSON：

```json
{
  "saved_at": "2026-03-04T17:03:07Z",
  "cookies": "abRequestId=xxx; webBuild=5.13.1; xsecappid=xhs-pc-web; a1=xxx; web_session=xxx; ..."
}
```

写入 `{output_path 所在目录}/.xhs-cookies.json`。

```
步骤 C：向用户确认
```

> 登录成功，Cookie 已保存到本地（下次执行时将自动恢复登录态）。开始执行搜索任务。

---

**推荐：持久化浏览器配置（最佳方案）**

上述 Cookie 文件方案受 `HttpOnly` 限制，恢复成功率约 60-70%。若希望 100% 免登录复用，推荐用户启动 Chrome 时指定固定的用户数据目录：

```bash
# macOS
/Applications/Google\ Chrome.app/Contents/MacOS/Google\ Chrome \
  --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.xhs-chrome-profile"

# Linux
google-chrome --remote-debugging-port=9222 \
  --user-data-dir="$HOME/.xhs-chrome-profile"
```

使用固定 `--user-data-dir` 后，Chrome 自身会持久化所有 Cookie（含 `HttpOnly`），下次启动自动恢复完整登录态，无需任何额外操作。

将此配置建议在首次检测到未登录时一并告知用户。

### 第一步：导航到搜索页

```
URL 模板：
https://www.xiaohongshu.com/search_result/?keyword={keyword_encoded}&type=51
```

使用浏览器导航到上述 URL，等待页面加载完成。`type=51` 表示搜索笔记。

### 第二步：应用筛选条件

URL 参数 `sort=popularity_descending&time=day` 不一定生效，必须通过 DOM 操作：

```javascript
// 1. 打开筛选面板
document.querySelector('div.filter').click();

// 2. 等待 500ms 后点击时间筛选项
// 可选值：'不限' | '一天内' | '一周内' | '半年内'
for (const el of document.querySelectorAll('span, div')) {
  if (el.textContent.trim() === '一天内' && el.offsetParent !== null) {
    el.click(); break;
  }
}

// 3. 等待 500ms 后点击排序方式
// 可选值：'综合' | '最多点赞' | '最新'
for (const el of document.querySelectorAll('span, div')) {
  if (el.textContent.trim() === '最多点赞' && el.offsetParent !== null) {
    el.click(); break;
  }
}

// 4. 等待 500ms 后收起筛选面板
for (const el of document.querySelectorAll('span, div')) {
  if (el.textContent.trim() === '收起' && el.offsetParent !== null) {
    el.click(); break;
  }
}
```

### 第三步：提取搜索结果列表

从 DOM 中提取所有笔记卡片的基础信息：

```javascript
const noteLinks = document.querySelectorAll('a[href*="/search_result/"]');
const notes = [];
const seen = new Set();

for (const link of noteLinks) {
  const href = link.getAttribute('href') || '';
  const idMatch = href.match(/\/search_result\/([a-f0-9]+)/);
  const tokenMatch = href.match(/xsec_token=([^&]+)/);

  if (idMatch && !seen.has(idMatch[1])) {
    seen.add(idMatch[1]);
    notes.push({
      id: idMatch[1],
      xsec_token: tokenMatch ? tokenMatch[1] : ''
    });
  }
}
```

**注意**：小红书搜索结果使用虚拟滚动，一次只渲染可视区域内的卡片。如需采集超过一屏的笔记，需要：
1. 提取当前可见笔记并保存
2. 滚动页面 `window.scrollTo(0, document.body.scrollHeight)`
3. 等待 2 秒后再次提取
4. 合并结果（按 `id` 去重）

### 第四步：逐个打开笔记获取详情

对每条笔记，点击搜索结果卡片打开笔记详情弹窗：

```javascript
const noteLink = document.querySelector(`a[href*="/search_result/${noteId}"]`);
noteLink.click();
```

等待 2 秒后，从弹窗中提取：

| 字段 | 来源 |
|:---|:---|
| 标题 | 弹窗内 `display_title` 文本 |
| 作者 | 弹窗内作者昵称链接文本 |
| 发布时间 | `corner_tag_info` 文本（如 `昨天 20:46`） |
| 点赞数 | 底部互动栏数字 |
| 收藏数 | 底部互动栏数字 |
| 评论数 | 底部互动栏数字 |

### 第五步：获取分享链接

**方法 A（推荐）：从搜索结果直接构造**

分享链接格式：

```
https://www.xiaohongshu.com/discovery/item/{noteId}?source=webshare&xhsshare=pc_web&xsec_token={xsec_token}&xsec_source=pc_share
```

其中 `noteId` 和 `xsec_token` 均在第三步已提取。此方法效率最高。

**方法 B：通过分享按钮获取（备用）**

如需获取完整分享文本（含短口令码），操作如下：

1. 在笔记详情弹窗中，找到并点击分享按钮：
   ```javascript
   document.querySelector('button.share-icon').click();
   ```
2. 等待分享面板弹出后，点击复制链接按钮（第一个按钮，图标为 `#ic_link`）：
   ```javascript
   const shareTools = document.querySelector('.share-tools');
   const buttons = shareTools.querySelectorAll('button');
   buttons[0].click(); // #ic_link 是复制链接按钮
   ```
3. 此操作会调用 API `POST /api/sns/web/share/code`，请求体为 `{"share_code":{"id":"{noteId}"}}`，返回短口令码
4. 同时将完整分享文本写入剪贴板，格式为：
   ```
   【{标题} - {作者} | 小红书】 😆 {口令码} 😆 {分享URL}
   ```

> **注意**：浏览器自动化环境下剪贴板拦截可能失败（`navigator.clipboard.writeText` 受安全策略限制），建议优先使用方法 A。

### 第六步：关闭弹窗并处理下一条

按 `Escape` 键或点击弹窗外部区域关闭当前笔记弹窗，返回搜索结果列表，继续处理下一条。

### 第七步：验证链接有效性

对构造的分享链接，随机抽取 1-2 条进行导航验证：
- 正常加载：URL 被重定向到 `/explore/{noteId}?...`，页面显示笔记内容
- 已删除：URL 被重定向到 `/404`，返回 `error_code=300031`

将已删除的笔记在输出中标记为「已删除」。

## 输出格式

生成 Markdown 文件，结构如下：

```markdown
# 小红书「{keyword}」热帖（{sort}）

> 搜索关键词：`{keyword}` | 筛选条件：**{time_filter}** + **{sort}** 排序
>
> 抓取日期：{YYYY-MM-DD} | 来源：[小红书](https://www.xiaohongshu.com/)

---

## 热帖排行

| 排名 | 标题 | 作者 | 发布时间 | 点赞 | 链接 |
|:---:|:---|:---|:---|---:|:---|
| 1 | {标题} | {作者} | {时间} | {点赞} | [查看]({分享链接}) |
| 2 | ... | ... | ... | ... | ... |

---

## 内容摘要

### 🔥 最热帖子：{标题}（{点赞}赞）
{简要总结}

### 分类汇总
- **类别A**：相关帖子摘要
- **类别B**：相关帖子摘要

---

## 相关搜索热词

| 热词 |
|:---|
| {从搜索页底部提取的相关搜索词} |
```

## 异常处理

| 异常场景 | 处理方式 |
|:---|:---|
| 页面加载超时 | 重试一次，仍失败则截图报告 |
| 筛选面板未弹出 | 等待 1 秒后重新点击 `div.filter` |
| 虚拟滚动丢失元素 | 滚动前保存已采集数据，滚动后增量采集 |
| 笔记弹窗未加载 | 等待 3 秒后重试，仍失败则跳过该笔记 |
| 笔记已删除（404） | 在表格中标记「已删除」，继续处理下一条 |
| 搜索无结果 | 通知用户更换关键词或放宽筛选条件 |
| 账号异常提示 | 通知用户手动刷新页面或重新登录 |

## 关键 API 参考

| API | 方法 | 用途 |
|:---|:---|:---|
| `/api/sns/web/v1/search/notes` | POST | 搜索笔记（需签名头 `x-s`, `x-t`） |
| `/api/sns/web/v1/feed` | POST | 获取笔记详情 |
| `/api/sns/web/share/code` | POST | 获取分享口令码 |
| `/api/sns/web/v2/comment/page` | GET | 获取评论列表 |

> 这些 API 均需要小红书的反爬签名头（`x-s`, `x-t`, `x-s-common`），无法直接通过 `fetch` 调用。必须通过浏览器页面内的正常交互触发。

## 注意事项

1. **登录状态**：必须确保浏览器已登录小红书，否则搜索结果受限
2. **操作节奏**：每次点击/操作之间至少间隔 500ms-2s，避免触发反爬
3. **xsec_token 时效性**：token 有时效限制，生成的分享链接建议尽快使用
4. **虚拟滚动**：搜索结果列表使用虚拟渲染，DOM 中同一时刻仅包含可视区域的卡片
5. **分享链接格式**：有效链接路径为 `/discovery/item/{id}`，而非 `/search_result/{id}` 或 `/explore/{id}`
