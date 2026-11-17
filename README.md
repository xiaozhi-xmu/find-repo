# find-repo

一个 Agent Skill：用 GitHub CLI (`gh`) 搜索、去重、评估并推荐开源仓库。

适用于「帮我找 GitHub 项目 / 仓库 / 开源方案 / CLI 工具」这类需求，尤其是当你给出功能约束（本地部署、免费、离线、无 API、指定语言）并需要在多个候选仓库之间做对比时。

## 它能做什么

- 根据需求生成 6 组英文搜索关键词（3 组核心词 + 3 组特色需求词）
- 用 `gh search repos` 按 star 排序抓取候选仓库
- 按 fullName 去重，并对高 star 项目进一步读取 README、最近提交做真实性验证
- 按 Star、活跃度、License、需求匹配度、社区健康度五个维度打分（总分 25）
- 输出仓库卡片：链接、clone 命令、语言、License、更新时间、README 摘要、仓库特色和注意事项
- 最后给出最终推荐、组合方案、快速开始命令和避坑提醒

## 前提

- 已安装 GitHub CLI：<https://cli.github.com/>
- 已登录：`gh auth status` 能正常返回账号

## 安装

把 `SKILL.md` 放进你所用客户端的 skills 目录即可，例如：

```bash
# Claude Code / 通用 agent skills 目录
mkdir -p ~/.claude/skills/find-repo
cp SKILL.md ~/.claude/skills/find-repo/

# Codex
mkdir -p ~/.codex/skills/find-repo
cp SKILL.md ~/.codex/skills/find-repo/
```

目录名建议保持 `find-repo`，与 SKILL.md frontmatter 中的 `name` 一致。

## 使用示例

直接描述需求即可触发，例如：

> 帮我找一个 Python 写的不依赖官方 API 的 Reddit 爬虫，要能本地跑。

Skill 会先展示生成的 6 组关键词供你校验方向，再给出评分排序后的候选清单和推荐结论。

## License

未指定。如需转载或二次分发，请先联系作者。
