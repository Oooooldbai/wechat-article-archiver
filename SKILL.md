---
name: wechat-article-archiver
description: |
  微信公众号文章自动抓取、去广告、双版本存档。
  使用 Python + BeautifulSoup 绕过微信反爬，提取完整正文。
  自动生成两份文件：原文存档(inbox) + 方法论提炼(topics)。
  支持半自动关键词映射流程。
  
  **触发时机（任一满足即加载）**：
  - 用户发送 mp.weixin.qq.com 链接
  - 用户说"微信文章存档"、"公众号文章入库"、"存档微信文章"、"保存公众号"
  - 用户说"看一下这篇文章"、"走入库流程"、"值得放知识库吗"
  - 用户说"这篇文章" + "入库" / "存档" / "提炼"
  
  **前置检查**：看到微信链接时，优先使用本 Skill 而非手动 Python 抓取。
license: MIT
compatibility: opencode
metadata:
  version: "0.1.1"
  openclaw:
    emoji: "📄"
    author: "女娲"
    created: "2026-04-24"
    updated: "2026-04-24"
---

# 微信文章存档器

## 使用场景

- 老白发来微信公众号文章链接，需要存档到知识库
- 有价值的行业文章/方法论文章需要双版本保存
- 需要移除广告和推广链接，保留核心内容

## 核心流程

### Step 1: 抓取文章

使用 Python + BeautifulSoup 绕过微信反爬：

\`\`\`python
import requests
from bs4 import BeautifulSoup

url = "https://mp.weixin.qq.com/s/xxxxx"
headers = {"User-Agent": "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36"}

resp = requests.get(url, headers=headers, timeout=15)
soup = BeautifulSoup(resp.content, "html.parser")

title = soup.select_one("#activity-name").get_text(strip=True)
author = soup.select_one("#js_name").get_text(strip=True)
content = soup.select_one("#js_content")
\`\`\`

### Step 2: 清理广告

必须移除的内容：
- 推广链接（包含 \`datawhale.psce\` 等）
- 二维码图片
- 报名/课程/扫码等 CTA
- "保姆级教程"等营销话术
- "点赞三连"等互动引导

保留的内容：
- 完整正文
- 核心方法论
- 案例和数据
- 作者信息

### Step 3: 生成双版本（命名规范）

**文件名格式**：\`YYYY-MM-DD-来源-描述-原文存档.md\`

- \`YYYY-MM-DD\`：当天日期
- \`来源\`：微信 / 知乎 / 公众号名称等
- \`描述\`：文章核心主题（简短）
- \`原文存档\`：固定后缀

#### 版本A：原文存档 (00-Inbox/)

文件名示例：\`2026-04-24-微信-扣子二创图文手账-原文存档.md\`

结构：
\`\`\`markdown
# 文章标题

**作者**: XXX  
**来源**: [原文链接](URL)  
**抓取时间**: YYYY-MM-DD

---

## 正文（广告/推广链接已移除）

[清理后的完整正文]

---

*本文档为微信文章原文存档，广告和无关链接已移除。*
\`\`\`

#### 版本B：方法论提炼 (20-Concepts/或topics/)

文件名：\`文章标题-方法论.md\`

结构：
\`\`\`markdown
# 文章标题 - 方法论提炼

> 来源: 作者《文章标题》  
> 原文存档: \`../00-Inbox/文章标题-原文存档.md\`

---

## 核心方法论

### 1. [方法名称]
[核心流程图解]

### 2. 关键机制
[分点说明]

### 3. 适用场景
1. 场景1
2. 场景2

### 4. 局限性
[客观列出不足]

---

## 与现有方法论对照

| 他们的方法 | 我们的方法 |
|-----------|-----------|
| ... | ... |

**启示**: [可借鉴的改进点]

---

**入库时间**: YYYY-MM-DD  
**提炼者**: 女娲
\`\`\`

### Step 4: 4步入库流程（强制检查清单）

**⚠️ 必须完成全部4步，缺一不可**

\`\`\`
□ 第1步：放入
   └── 00-Inbox/YYYY-MM-DD-来源-描述-原文存档.md
   └── 20-Concepts/（或对应topics目录）/文章标题-方法论.md

□ 第2步：Git commit + push
   └── cd /root/Laobai-Second-Brain
   └── git add -A
   └── git commit -m "Add: 文章标题（原文+提炼）"
   └── git push origin main

□ 第3步：更新索引
   └── 编辑 /root/.openclaw/workspace/memory/index.md
   └── 添加关键词映射（至少3-5个关键词）
   └── cd /root/.openclaw/workspace
   └── git add -A && git commit -m "Update index.md: 添加xxx关键词"
   └── git push origin master

□ 第4步：写日志
   └── 编辑 /root/Laobai-Second-Brain/05-Agent-Rules/Daily-Logs/YYYY-MM-DD-nuwa.md
   └── 记录：入库文件、来源、关键词、状态
\`\`\`

**自检问题**：
- [ ] 文件名是否符合 \`YYYY-MM-DD-来源-描述-原文存档.md\` 格式？
- [ ] 是否 commit 到 Laobai-Second-Brain？
- [ ] 是否更新了 memory/index.md？
- [ ] 是否写了 Daily-Logs？
- [ ] 两个仓库都 push 了吗？

### Step 5: 半自动关键词映射

生成建议关键词，等待用户确认：

\`\`\`
文件：00-Inbox/文章标题-原文存档.md
建议关键词：
- "关键词1" / "关键词2" / "关键词3"
- "关键词4" / "关键词5"
对应路径：Laobai-Second-Brain/00-Inbox/

文件：20-Concepts/ai-workflows/文章标题-方法论.md
建议关键词：
- "方法论关键词1" / "方法论关键词2"
对应路径：Laobai-Second-Brain/20-Concepts/ai-workflows/
\`\`\`

用户回复后执行写入。

## 文档模板

### 原文存档模板

\`\`\`markdown
# [文章标题]

**作者**: [作者名]  
**来源**: [原文链接]  
**抓取时间**: YYYY-MM-DD

---

## 正文（广告/推广链接已移除）

[清理后的正文，段落之间空一行]

---

*本文档为微信文章原文存档，广告和无关链接已移除，保留核心方法论内容。*
\`\`\`

### 方法论提炼模板

\`\`\`markdown
# [文章标题] - 方法论提炼

> 来源: [作者]《[文章标题]》  
> 原文存档: \`../00-Inbox/[文件名]\`

---

## 核心方法论

### 技术栈/工具
[使用的工具或方法]

### 核心流程
\`\`\`
步骤1 → 步骤2 → 步骤3
\`\`\`

### 关键机制
| 机制 | 说明 |
|------|------|
| 机制1 | 说明1 |
| 机制2 | 说明2 |

### 适用场景
1. 场景描述
2. 场景描述

### 局限性
- 局限性1
- 局限性2

---

## 与现有方法论对照

| 他们的方法 | 我们的方法 |
|-----------|-----------|
| ... | ... |

**启示**: [可借鉴点]

---

**入库时间**: YYYY-MM-DD  
**提炼者**: 女娲
\`\`\`

## 示例：完整执行流程

**用户输入**: "https://mp.weixin.qq.com/s/xxx 看看这个文章，值得放知识库吗？"

**Skill 执行**:
1. 抓取文章 → Python + BS4
2. 清理广告 → 移除推广链接和CTA
3. 生成双版本：
   - \`00-Inbox/Kimi-K2.6-Hermes-LLM-Wiki-原文存档.md\`
   - \`20-Concepts/ai-workflows/LLM-Wiki-Hermes-方法论.md\`
4. Git commit + push
5. 生成关键词建议：
   - "LLM Wiki" / "Hermes" / "Kimi K2.6"
   - "知识库自动化" / "SCHEMA.md" / "Skill沉淀"
6. 用户回复 "可以"
7. 写入 index.md
8. 记录到 Daily-Logs
9. 报告完成

## 使用限制

- 仅用于存档和学习目的
- 尊重原作者版权
- 不用于商业分发

## 更新记录

| 版本 | 日期 | 更新内容 |
|------|------|---------|
| 0.1.1 | 2026-04-24 | 强化4步流程：添加命名规范、强制检查清单、自检问题 |
| 0.1.0 | 2026-04-24 | 初始版本，支持双版本存档+半自动关键词映射 |
