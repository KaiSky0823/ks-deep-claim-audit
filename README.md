# ks-deep-claim-audit

**网络宣称深度核查 / Deep Claim Audit** — a Claude Code skill.

> 别人在 X/文章里吹一个 GitHub 项目、AI 工具、产品、或某个数据/性能/热度数字。
> 这个 skill 让 Claude **不附和、不预设**,把每条具体声称对到源头证据,**客观判定是 真实 / 部分真实 / 夸大 / 虚假 / 无法证实**,最后出一份可追溯的 `.md` 核查报告。

不是"夸它"也不是"踩它"——**摘掉宣传滤镜,只让可追溯的证据说话**。

![License](https://img.shields.io/badge/License-MIT-blue.svg) ![Claude Code](https://img.shields.io/badge/Claude%20Code-skill-orange)

---

## 它解决什么痛点

信息流里全是「最/第一/快 N 倍/N 万星/Reddit 炸了/省几万/替代 XX/觉醒」式的技术宣传。它们**部分真、部分水**,而人脑(和顺着兴奋走的 AI)很容易被营销框架带偏。这个 skill 把"做尽职调查"这件事**流程化、可复用**:

- **逐条判真伪**,不是给个模糊好评/差评;
- **证据可追溯**(代码带 `文件:行号`,数字带 `命令输出 / URL+日期`);
- **客观中立**,主动点明"宣传在哪带偏你";
- 沉淀了大量**真实踩过的坑**(见下),不靠临场发挥。

## 跟同类的边界(避免用错)

| 你想干嘛 | 用这个吗 |
|---|---|
| 核查【别人】吹的某个 repo/工具/产品/数据是不是真的 | ✅ **本 skill** |
| 核查【你自己写的】文章稿有没有事实错误 | ❌ 用 `article-fact-check` |
| 想要某个开放主题的多源研究报告 | ❌ 用 `deep-research` |
| 想精读理解一篇论文/文章的思想 | ❌ 用 `deep-reading-analyst` |

## 安装

把本仓的 `SKILL.md` 放到任一处(Claude Code 自动识别):

```bash
# 用户级(所有 session 通用)
mkdir -p ~/.claude/skills/ks-deep-claim-audit
curl -fsSL -o ~/.claude/skills/ks-deep-claim-audit/SKILL.md \
  https://raw.githubusercontent.com/KaiSky0823/ks-deep-claim-audit/main/SKILL.md

# 或 项目级
mkdir -p <你的项目>/.claude/skills/ks-deep-claim-audit
cp SKILL.md <你的项目>/.claude/skills/ks-deep-claim-audit/
```

## 怎么用

贴一段推文/文章/截图/repo 链接 + 一句话,即自动激活:

- 「帮我**深度地毯式核查**一下这个项目,可以用 dynamic workflow」
- 「这个 GitHub / 工具 **靠谱吗 / 是不是真的 / 是不是夸大 / 值不值得用**」
- 「这个 benchmark·数据·性能数字**成立吗 / 有没有水分**」
- 「这条推说某项目 N 万星,**地毯式核一下**」

**可选**:加一句「重点查它的 **基准是否公平 / 安全能不能装 / 作者靠不靠谱 / 星是不是刷的**」聚焦;说「轻量核一下」或「大胆多派 agent」控制力度。

## 它背后怎么跑(5 步 SOP)

1. **侦察事实底座**:`gh api`(stars/forks/license/commits)、克隆**只读**审查、查 PyPI/npm/Maven、看 README——先摸清地基再决定派谁。
2. **派 dynamic Workflow 扇出**:多个 Sonnet agent 按维度并行,各自输出统一 schema(`claim / verdict / evidence / confidence / flag`)。
3. **亲自交叉核对载重结论**:最高风险/最反直觉/agent 间矛盾的几条,自己跑源头(不信自报),用实测纠正纸面判断。
4. **客观综合**:出逐条 verdict 的报告。
5. **收尾**:存档,必要时排复查提醒。

## 输出长什么样

一份 `.md` 报告,固定结构:**一句话结论(去滤镜)→ 逐句核查表(每条 `真实/部分真实/夸大/虚假/⚫无法证实` + 证据)→ 客观画像 → 维度展开 → 给你的实用建议(分人群)→ 方法与透明纠错**。

## 核查维度(按对象裁剪)

声称↔代码/事实 · 安全可用性 · 基准/性能公平性 · 社媒/热度虚实 · 作者可信度/变现 · 概念来历/原创性 · 合规法律。

## 铁律

- **客观,不 mirror 宣传语气**;结论由证据定。
- **不信自报**:agent 报的、宣传说的、自己上轮的判断,关键几条亲验源头;错了透明纠正。
- **只读审查、绝不跑不可信代码**;`curl|bash` 类安装交还用户自己执行。
- **取不到真值** → 重试/换源/找间接证据 → 仍不行就标 `⚫无法证实`,**绝不软化成"可能/大概率"**塞进主结论。
- 研究/核对类 agent 用 **Sonnet**,深度综合与判断用 **Opus** 收口(成本纪律)。

## 沉淀的真实坑(为什么它比临场更稳)

浅克隆只 1 条 git 历史(commit 史要远端核)· 数据真假(真 API vs 硬编码/随机模拟/僵尸功能)· 基准硬件公平性(GPU 引擎被拿 CPU 跑残)· 自跑基准 vs 独立榜 · "最/第一"门槛因领域过新而极低 · stars:watchers 异常比 · AI 批量代工识别(commit 邮箱全 AI CLI + 残留工作流文件)· 地理封锁端点找替代 · 演示站/域名安全评分 · 远程指令链(`Read <URL> and follow`)· 关联交易/sell-the-news · 概念命名者归属。

## License

[MIT](LICENSE) — 自由使用、复制、修改、分发(含商用),署名欢迎但不强制。

---

> 由 [Claude Code](https://claude.com/claude-code) 协助蒸馏自多轮真实核查实跑。
