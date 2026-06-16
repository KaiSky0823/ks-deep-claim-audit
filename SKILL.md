---
name: ks-deep-claim-audit
description: >-
  对【别人】在网上(X 推文 / 文章 / 截图 / 营销话 / 朋友转发)宣称的某个 GitHub 开源项目 · AI 工具 · 产品 · 数据/性能/热度,
  做毫无遗漏的深度地毯式核查与尽职调查——客观判定每条声称是 真实/部分真实/夸大/虚假/无法证实,而不是附和宣传。
  触发:用户贴来【别人的】宣传/推文/截图/repo 链接要求核真伪,或要求「深度地毯式 挖掘/核实/审查/验证/分析/判断」「毫无遗漏」
  「可以大胆用 dynamic workflow / agent teams」;或直接问「这个项目/工具/repo 靠谱吗 / 是不是真的 / 是不是夸大 /
  值不值得用 / 这个推文可信吗 / 帮我查查这个 GitHub / 这个 benchmark·数据·性能数字成立吗 / 这个声称有没有水分」。
  不触发:① 核查【用户自己写的文章稿】→ article-fact-check;② 开放主题的多源研究报告(非核查特定宣称)→ deep-research;
  ③ 精读理解一篇论文/文章的思想 → deep-reading-analyst。
  核心动作:先自己侦察事实底座 → 派 Workflow 多 Sonnet agent 按维度扇出 → 亲自交叉核对载重结论 → 客观出逐条 verdict 的 .md 报告。
  触发词:深度核查、地毯式、毫无遗漏、挖掘洞察、核实验证、是不是真的、是不是夸大、靠谱吗、可信吗、查查这个项目、
  推文核查、X 博主、宣称、尽调、due diligence、fact-check this repo/tool/claim、verify this hype。
---

# Deep Claim Audit · 网络宣称深度核查

> **别人在网上吹一个项目/工具/产品/数据,你不附和、不预设,把每条具体声称对到源头证据,客观判真伪夸大,出一份可追溯的 .md 报告。** 不是"夸它"也不是"踩它"——摘掉宣传滤镜,只让证据说话。

来源:从本项目多轮实跑蒸馏(Osiris / OpenDataLoader / 股票数据 skill / loop-engineering 橙皮书 / 觅游 Meyo)。配套:`agent-team-review`(核验纪律)、`agent-team-collaboration`(派活分工/分模型)、记忆 `agent-model-by-complexity`。

---

## 铁律(任何时候不可违反)

1. **客观,不 mirror 宣传语气**:不跟博主/用户的兴奋走,不预设真假,呈现平衡,**主动点明"宣传框架在哪带偏你"**。结论可以是"真东西"也可以是"夸大/骗局",由证据定。
2. **证据可追溯到源头**:代码结论带 `文件:行号`;数字/热度带 `命令输出 或 URL+日期`;不用二手看板、不编数。
   **取不到真值 → 三级处理**:① 换路径重试(镜像/替代端点/`gh api` 远端代本地/Wayback `web.archive.org`/archive.today)、记录重试;② 找间接证据(注明"间接推断"+来源可信度);③ 在逐句核查表里以 **`⚫ 无法证实`** 写死,并在"方法与局限"说明取不到的原因与对结论的影响。**绝不**把"无法证实"悄悄软化成"可能/应该/大概率"塞进主结论。
3. **不信自报**(`agent-team-review`):agent 报的、宣传说的、自己上一轮的判断,**最高风险/最反直觉/agent 间矛盾的那几条,亲自跑源头复核**;矛盾由主循环(Opus)裁决;错了 → **透明纠正,写进报告**。
4. **只读审查、绝不跑不可信代码**:克隆只许 Read/Grep 静态看;**绝不运行**安装/构建/启动脚本(`npm install`/`run`、`docker`、`node`、`python setup`…)。`curl|bash` 类外部安装**不由审查者执行**——把命令原文贴给用户,注明"需你在自己机器上确认后运行,我不代跑"。读源码找风险但不执行,**安全静态速查**:
   ```bash
   grep -rEn 'postinstall|preinstall|prepare' package.json          # 装包即执行钩子
   grep -rEn 'eval\(|Function\(|atob\(|base64' <repo>/src           # 混淆
   grep -rEn 'fetch\(|axios|requests\.(get|post)' <repo>/src | grep -vE 'localhost|127\.0'  # 外域请求(查有没有发去作者域)
   grep -rEn 'coinhive|miner|stratum' <repo>/                       # 挖矿
   ```
5. **成本纪律**(`agent-model-by-complexity`):扇出的检索/取数/核对类 agent 用 `model: 'sonnet'`;深度综合与最终判断由主循环(Opus)收口。**思考强度是会话级,不能 per-agent 设。**
6. **报告存 `.md`**:命名 `<对象>-深度核查报告.md`,存本项目目录。

---

## SOP

### STEP 0 · 定位对象与声称
抽出 **① 对象**(哪个 repo/工具/产品/站)+ **② 每条可验证的具体声称**(逐条列、标谁说的)。声称类型:功能/数据真假、性能基准("快 N 倍/第一/0.9xx")、热度("Reddit 炸了/N 万星")、价值("省几万/政府级/替代 XX")、来历("最早/最系统/原创")、安全/开源/免费。

### STEP 1 · 先自己侦察"事实底座"(scout 出 work-list 再 orchestrate,别一上来就派 agent)
- **GitHub**:`gh api repos/<o>/<r>`(stars/forks/created/pushed/license/language/size/archived)+ `gh api users/<o>`(账号年龄/粉丝/repos)+ README + `gh api .../commits?per_page=100` + contributors。
- **stars:watchers 比**:`gh api repos/<o>/<r> --jq '[.stargazers_count,.watchers_count]'`。正常热门约 20-80:1;**>200:1 是异常**(无人长期关注 + 短期暴涨星 = 刷量特征)。
- **AI 批量代工识别**:`gh api .../commits?per_page=100 --jq '[.[].commit.author.email]'` 若全是 `gemini@example.com`/`claude@…`/`copilot-swe-agent` 等 AI CLI 默认邮箱 = AI 代写;`grep -rEn 'DO_NOT_PUSH|AGENTS\.md|implementation_plan|C:/Users/|/home/' <repo>` 找 AI 工作流残留 + 泄露的开发机路径。人工 commit <10 次 → 报告标"AI 代工生成"而非"作者建站"。
- **克隆只读**:`git clone --depth 1 <url> /tmp/<x>-audit`。⚠️ **浅克隆只有 1 条 git 历史**(踩过的坑)——commit 史一律 `gh api .../commits` 远端核,别据本地误判"某 commit 不存在"。
- **包真实性**:`curl -s https://pypi.org/pypi/<pkg>/json` / npm / Maven——版本、是否存在、依赖(extra 里藏了什么)。
- **是文档还是代码**:`language: None` + 体积小 + 全是 .md → 多半是"给 AI 读的 skill/指南/橙皮书",不是能跑的软件(别被"完整系统/数据仓库"框住)。
- **地理封锁排查**:`curl -I <端点/演示站>` 返回 000/403/超时 → 先区分 geo-block vs 真挂了 vs 不存在(换 `-x 代理` / `--resolve` 测 CDN / `curl -L` 看重定向);找 README/issue 有无替代端点(如 `push2`→`push2delay`)。区分不清就标"网络受限,此项未核实",别直接判"不可用"。
- **演示站/作者域名安全**:有演示站或关联商业域名时,WebSearch 查 ScamAdviser / Gridinsoft 评分 + `whois` 注册时间(仓库发布后几天才注册 = 红旗)。

### STEP 2 · 派 dynamic Workflow 扇出(Sonnet)
用 `Workflow` 工具 `parallel()` 派多个 Sonnet agent(本环境无 Workflow 时用 `Agent`/`Task` 并行),每个输出**统一 schema**:
```json
{"claim":"<被核查的原话>","verdict":"真实|部分真实·有前提|夸大|虚假·误导|无法证实",
 "evidence":"<文件:行号 / URL+日期 / 命令输出>","confidence":"高|中|低","flag":"<需主循环复核就写原因,否则空>"}
```
> 在 Workflow 脚本里字面量 `Math`+`.random` 会触发确定性检查 → 用变量拼接绕开(`const RND='Math'+'.random'`)。

**编队规模(scout 完后定)**:单 repo/单工具/声称≤10 条 → 3-5 路;多仓/多产品/声称>10 条/跨语言市场 → 8-12 路(先用 `agent-team-collaboration` 规划分工防维度重叠);每多一个营销夸大信号(最/第一/快N倍/炸了/省几万/发币)就加一路对应维度。

**维度菜单**(按对象取 3-5 个):

| 维度 | 查什么 |
|---|---|
| **声称↔代码/事实** | 逐条对到源码/数据:真接外部 API vs 硬编码/`Math` 随机模拟/僵尸功能(UI 有按钮、后端路由不存在);数据源标注是否与代码一致 |
| **安全 / 可用性** | 装包钩子/外域 exfil/混淆/挖矿/非 root(见铁律 4 速查);开箱即用还是要另起后端/密钥;**agent 类产品额外查**:有无 `Read <外部URL> and follow instructions` 类远程指令链(链有多深、运营方能否随时改写、用户可否审计、有无写凭证/定时任务) |
| **基准 / 性能公平性** | 自跑还是独立榜(OmniDocBench 等);对比公平吗(同硬件? GPU 引擎被拿 CPU 跑残? 同模式?);**对手代码是否在基准仓内可独立复跑、ground-truth 标注方/流程是否披露、语料是否代表真实场景**——自建+不可复现+未披露=自家擂台,独立可信度低;"最/第一"门槛是否因领域过新而极低 |
| **社媒 / 热度虚实** | "Reddit 炸了/N 万星":找得到原帖吗?star 速度/曲线/刷星迹象、stars:watchers 比、克隆仓刷 fork、有没有真上 Trending、营销号转 vs 自发、演示站真假 |
| **作者 / 可信度 / 变现** | 作者背景(批量 AI 建站? 真 builder? 机构背书?)、是否自我推广(吹自己作品)、变现路径(打赏/付费墙/课程/发币)、灰色/诈骗记录、关联域名安全评分(STEP 1) |
| **概念 / 来历 / 原创性** | 术语是自创还是已有被命名(查源头人/首发时间)、核心框架原创还是二手整合、"全新" vs "旧实践被重命名" |
| **合规 / 法律** | 抓取是否违 ToS、数据授权、许可证传染(GPL/AGPL)、"商用没顾虑"是否成立 |

**维度选取速查(按对象类型)**:

| 对象 | 必选 | 加选 |
|---|---|---|
| GitHub 工具/库(有性能声称) | 声称↔代码 · 基准公平性 · 安全可用性 | 热度(若"N万星/炸了")· 作者(若个人项目) |
| GitHub 文档/橙皮书/skill | 声称↔事实 · 概念来历原创性 | 作者(若自我推广) |
| X 推文/纯热度核查(无 repo) | 社媒热度虚实 · 作者可信度变现 | 声称↔事实(若有具体数据) |
| 产品/SaaS/付费工具 | 声称↔事实 · 安全可用性 · 合规法律 | 作者变现(若有课程/发币) |
| 含"发币/ICO/空投" | 作者变现 · 合规法律 · 社媒热度 | 关联域名安全评分 |

### STEP 3 · 亲自交叉核对载重结论(不信 agent 自报)
挑出 `flag` 非空、`confidence:低`、或最高风险/最反直觉/agent 间矛盾的 2-5 条,**自己跑源头**:读那段代码、`curl` 那个端点看真回什么、`gh api` 那个数字、WebFetch 那个原帖。用实测纠正纸面判断;自己之前看走眼也当场认、当场改。

### STEP 4 · 客观综合,出 .md 报告
**报告头(必备)**:
```
> 核查日期:YYYY-MM-DD | 对象:<项目/工具/推文> | 方法:<几路 agent + 几维 + 有无亲测交叉核> | 立场:客观中立。
```
**正文骨架(可直接套)**:
```markdown
# <对象> —— 深度核查报告
## 一句话结论
**<去滤镜定性:真东西/夸大/骗局,一句说死,带最关键的限定>**
## 一、逐句核查
| 原话/声称 | 判定 | 依据(文件:行号 / URL+日期) |
|---|---|---|
| … | **真实 / 部分真实·有前提 / 夸大 / 虚假·误导 / ⚫无法证实** | … |
## 二、它到底是什么(客观画像)
## 三、<核心争议维度展开>(数据真假/基准公平性/热度虚实/安全/作者/来历…)
## 四、给你的实用建议(分层:如 业余玩家可用 / 严肃生产别用)
## 附:方法与透明纠错
- 证据级别(代码=本地只读克隆带行号;数字=命令输出;社媒=Web 检索+日期)
- agent 误判纠正(哪条被复核推翻、原因、纠正后结论)
- 未做的事(哪些运行时行为未亲测、对结论的影响)
```

### STEP 5 · 收尾
报告路径告诉用户;涉及未落地的后续(等某事件/财报再复查)按需提议 `/schedule`;值得长期记的事实写记忆。
> **与 agent-team-review 边界**:STEP 3 已内嵌其核心纪律(自跑源头/主循环裁决矛盾/透明纠错),无需显式调用;仅当扇出 ≥6 路或出现两路结论完全相反时,可用 `/agent-team-review` 做专项交叉核对再汇入。

---

## 适用 / 不适用
- **适用**:别人在网上对某 repo/工具/产品/数据/性能/热度的**宣称**要核真伪,尤其带营销夸张("最/第一/快N倍/炸了/省几万/觉醒/替代")的。
- **不适用**:核查用户**自己写的文章稿**(`article-fact-check`);开放式主题研究(`deep-research`);论文精读(`deep-reading-analyst`)。
