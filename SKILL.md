---
name: find-repo
version: 1.0.0
description: 用 GitHub CLI 搜索并评估开源仓库。适用于用户要求“帮我找 GitHub 项目/仓库/开源方案/CLI 工具”，尤其当需求包含功能约束、技术栈、离线/本地部署/免费/无 API 等筛选条件时。
---

# Find Repo

使用 `gh` 在 GitHub 上搜索、去重、评估并推荐候选仓库。

## 何时使用

- 用户要求“找 GitHub 项目 / 仓库 / 开源方案 / CLI 工具”
- 用户描述一个功能，希望你找到可复用的开源实现
- 用户给出约束条件，例如“本地部署”“免费”“无 API”“离线”“指定语言”
- 需要在多个候选仓库之间做对比和推荐

## 前提

- 优先使用 GitHub，而不是普通搜索引擎
- 本 skill 默认依赖已安装并已登录的 `gh`
- 搜索关键词一律使用英文

## 工作流程

### 第一步：分析需求并生成 6 组搜索关键词

根据用户描述，严格生成 6 组英文关键词：

- 3 组核心通用词
- 3 组特色需求词

规则：

- 核心通用词只描述“做什么”，不带限定条件
- 特色需求词必须同时包含核心功能和用户约束
- 每组词从不同角度切入，例如技术术语、功能表达、数据格式、产品形态

示例：

- 用户说“Python Reddit 爬虫，不需要官方 API”
- 核心通用词：`reddit scraper`、`reddit crawler`、`reddit data extraction`
- 特色需求词：`reddit scraper without api`、`reddit unofficial api`、`reddit scraping no auth`

### 第二步：执行搜索

对每组关键词执行：

```bash
gh search repos "{关键词}" --sort=stars --limit=5 --json name,fullName,url,description,stargazersCount,updatedAt,language,license
```

如果用户指定了语言，追加：

```bash
--language "{语言}"
```

### 第三步：去重并深度评估

合并 6 组结果并按 `fullName` 去重，然后按 star 数量降序排列。

评估范围：

1. 所有 `star >= 500` 的仓库
2. `star 100-499` 的前 15 个
3. `star < 100` 的前 10 个

终止条件满足任一即可：

1. 已找到 5 个明确符合需求的仓库
2. 已评估 25 个仓库仍未找到足够候选
3. 已遍历所有去重后的仓库

不要只看 description。高 star 项目必须进一步读 README 验证。

对每个候选仓库执行：

1. 基础信息

```bash
gh api repos/{owner}/{repo} --jq '{stars: .stargazers_count, forks: .forks_count, open_issues: .open_issues_count, updated: .updated_at, created: .created_at, license: .license.spdx_id, archived: .archived, description: .description, topics: .topics, clone_url: .clone_url, homepage: .homepage}'
```

2. README 验证

```bash
gh api repos/{owner}/{repo}/readme --jq '.content' 2>$null | base64 -d 2>$null
```

如果 README 很长，只取前 200 行做分析。

3. 最近提交

```bash
gh api repos/{owner}/{repo}/commits --jq '.[0:5] | .[] | {date: .commit.author.date, message: .commit.message}' 2>$null
```

记录匹配状态：

- `✅ 符合`
- `❌ 不符合`
- `❓ 不确定`

### 第四步：评分

对符合需求的仓库按 5 个维度各打 1-5 分，总分 25 分。

1. Star 数量
- `> 5k = 5`
- `> 2k = 4`
- `> 1k = 3`
- `> 500 = 2`
- `> 100 = 1`

2. 活跃度
- 1 个月内更新 = 5
- 3 个月内更新 = 4
- 6 个月内更新 = 3
- 1 年内更新 = 2
- 超过 1 年 = 1

3. License 友好度
- MIT / Apache / BSD = 5
- LGPL = 3
- GPL = 2
- 无 license = 1

4. 需求匹配度
- 5：README 明确满足核心需求且功能完整
- 4：README 满足核心需求但存在限制
- 3：需要进一步读代码确认
- 2：README 模糊
- 1：明确不符合

5. 社区健康度
- 综合 forks、open issues、是否 archived 判断

### 第五步：输出结果

按总分排序，对每个推荐仓库输出：

```text
排名 x：仓库名
仓库: owner/repo
链接: https://github.com/owner/repo
Clone: git clone https://github.com/owner/repo.git
Star: 数量
语言: Python / TypeScript / ...
License: MIT / Apache-2.0 / ...
最后更新: 日期
总分: xx / 25

README 摘要：
用 2-3 句话说明项目做什么、核心能力、如何使用。

仓库特色：
- 特色 1
- 特色 2
- 特色 3

注意事项：
说明依赖、维护状态、学习成本、文档质量等潜在问题。
```

### 第六步：总结建议

在仓库卡片之后，补充：

1. 最终推荐
2. 组合方案
3. 快速开始命令
4. 避坑提醒

## 注意事项

- 如果某个命令失败，继续评估其他候选
- 如果结果太少，再扩一轮更宽泛的英文关键词
- 如果用户没指定语言，不限制语言
- 优先推荐最近 6 个月内更新的仓库
- README 摘要与结论用中文输出
- Clone URL 必须给完整可执行命令
- 仓库特色必须基于 README 或仓库元数据，不要泛泛而谈

## 输出要求

- 先展示你生成的 6 组关键词，方便用户校验搜索方向
- 明确说明你最终选择了哪些仓库进入深度评估
- 如果存在歧义仓库，说明为什么排除
- 用户问“你说的是哪个项目”时，明确给出 `owner/repo`

