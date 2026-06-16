# ks-deep-claim-audit

A Claude Code skill · 网络宣称深度核查 / Deep Claim Audit

对【别人】在网上(X 推文 / 文章 / 截图 / 营销话)宣称的某个 **GitHub 开源项目 · AI 工具 · 产品 · 数据/性能/热度**,做毫无遗漏的深度地毯式核查与尽职调查——客观判定每条声称是 **真实 / 部分真实 / 夸大 / 虚假 / 无法证实**,而不是附和宣传。

> 不是"夸它"也不是"踩它"——摘掉宣传滤镜,只让可追溯的证据说话。

## 怎么装

把 `SKILL.md` 放到任一处(Claude Code 自动识别):

- 用户级(所有 session 通用):`~/.claude/skills/ks-deep-claim-audit/SKILL.md`
- 项目级:`<你的项目>/.claude/skills/ks-deep-claim-audit/SKILL.md`

## 怎么用

贴一段推文/文章/截图/repo 链接 + 一句话即可自动激活:

- 「帮我**深度地毯式核查**一下这个项目,可以用 dynamic workflow」
- 「这个 GitHub / 工具 **靠谱吗 / 是不是真的 / 是不是夸大**」
- 「这个 benchmark·数据·性能数字**成立吗 / 有没有水分**」

它会:**先侦察事实底座(gh api · 克隆只读 · 查包)→ 派多个 Sonnet agent 按维度并行扇出 → 亲自交叉核对载重结论 → 客观出逐条 verdict 的 `.md` 报告**。

详细方法论见 [`SKILL.md`](SKILL.md)。

## License

MIT
