# 两子技能审阅汇总 — writing-for-agents 框架

- 审阅对象：`skills/cognition/`、`skills/foundation/`
- 框架：Matt Pocock `writing-for-agents`（`SKILL.md` + `SKILL-MECHANICS.md`）
- 方式：两个并行审阅 agent，各自独立读完框架与目标技能全部文件，彼此不可见；本汇总在两份独立结论之上做交叉比对
- 纪律：全程只读，未改动 `skills/` 下任何文件
- 明细：`writing-for-agents-审阅-cognition.md`、`writing-for-agents-审阅-foundation.md`

---

## 1. 总体判断

两个技能在结构上都已可用（darwin 基线 76.5 / 77.0，两轮迭代已清掉 dim3「异常兜底」与 dim4「检查点」），但换到 writing-for-agents 这把尺子下，暴露的是**同一类病**：完成判据留了「无需做功即可满足」的口子，常驻指针里缓存了正文的身份。

二者的头号问题并不相同，但同源：

| | 头号问题 | 失效机制 |
|---|---|---|
| cognition | **防赶工防御整体失效** | 抵抗早停的希望押在两处 CHECKPOINT 上，而它们是内联确认、不是上下文边界；真正能抵挡的完成判据又被「或一条明确的检索无果」击穿，并经兜底原则自动豁免到下一步 |
| foundation | **执行会走偏，且走偏后仍判绿** | 「模板」一词在指针层未消歧，同一裸词指两个文件；叠加「无变化」可一键满足的完成判据，最可能用错基准，还把没做的事汇报成做完 |

一句话：**cognition 的问题让流程跑得快而浅，foundation 的问题让流程跑错方向而自以为对。**

---

## 2. 共性缺陷（跨技能同源，建议优先修）

### 模式 A · 完成判据含「免费逃逸项」（demand = 0）★ 最高优先

| | 证据 |
|---|---|
| cognition | `SKILL.md:70`「每个待答问题都有至少一条可追溯来源，**或一条明确的「检索无果」**」；`SKILL.md:104` 又把该节点在第三步的来源要求**自动豁免**——可先标无果、再写无来源的反转 |
| foundation | `SKILL.md:83`「所处理文档的全部章节处置有交代」，四选一含「无变化」而它**无举证义务**——每节一律标「无变化」即判据全绿 |

框架依据：`Steps and completion criteria` — 判据须 both checkable and exhaustive；Demand 决定 legwork。两份报告独立指向同一处：**判据可检查，但可被一个 token 满足，拉力为零。**

统一改法：逃逸项一律附加举证义务；无据的节点只留白、不产出结论。foundation 的 `SKILL.md:105`（同步判据）是本仓库内**最强的样板**——「每个模板没有的标题与表头，都有一条处置……漏一个即未完成」，`L83` 应改写到它的强度。

### 模式 B · description 缓存正文身份 / 一个分支写两遍

- cognition `L3`：「九维」由 `L30-44` 定义、「三件套」由 `L10-18` 定义、「反转」由 `L22-28` 定义——指针里三样全是正文复述；触发段「深挖某个实体」与「构建/生成知识深度树文档」是**一个分支写了两遍**。
- foundation `L3`：操作名（初始化/更新/同步）先列一次、释义句再列一次；结尾「即使用户没有说出技能名或完整文档名」不改变任何路径，是纯 no-op。

框架依据：`Context pointers` — Cut identity the body already carries / One trigger per branch；`Pruning` — no-op。description 是唯一常驻上下文，每轮付费。两份报告都已给出可直接替换的整行文案。

### 模式 C · CHECKPOINT 被当成上下文边界

- cognition `L59`、`L85`；foundation `L63`、`L98`、`L102`。
- 框架依据：`Steps and completion criteria` —「Hiding only works across a real context boundary (a hand-off or a subagent dispatch; an inline call leaves the later steps in context and clears nothing)」。
- **两份报告结论一致**：这些检查点作为**人类审批门**是对的（`The two loads`：认知负载「is the price of human agency」），要改的是标签与预期（删 `🛑 STOP` 的边界暗示），把抵抗力补回判据；**都不要为此拆分技能**——框架规定先 sharpen the bound，且拆分要再花一份常驻负载。

### 模式 D · 单一事实源被破坏（重复）

- cognition：「反转」的规则碎在五处（`L24-28` / `L76-78` / `L81` / `L98` / `domains.md:50-66`）；十一个领域名单双份（`L53` vs `domains.md:7-19`，且 SKILL.md 那份是残缺版）；`domains.md` 的「反转主色调」被 `L28` 与 `L47` 指了两次。
- foundation：「不碰其他文档」这条禁令写了三处（`L13` / `L116` / `L124`）；授权旁路写了两遍（`L63` / `L98`）；「禁止依赖方向」这条具体规则在 `L45` 与 `templates/AGENTS.md:38` 各说一遍。

框架依据：`Pruning` — single source of truth，改行为的代价应是「一处编辑」。

### 模式 E · 指针未编码触发条件

- cognition `L46-47`：两行只罗列「里面有什么」，而**何时去读**散在 `L54`/`L63`/`L78`——指针与触发分离。
- foundation `L19-20`：明确立了两个 `templates/`（技能自带＝只读基准，仓库内＝写入目标），但 `L81`/`L101`/`L105`/`L117` 全部退化为裸词「模板」——同一词两个指称。这是 foundation 的最高严重度发现。

框架依据：`Context pointers` —「A context pointer … **encodes the condition for reaching it**」；「The pointer's *wording*, not its target, decides when the agent reaches the material」。

### 模式 F · 靠否定表达

- cognition `L108`「不编造」、`L110`「不为单个实体新造维度」、`L79`「而非虚构示例」；foundation `L13` 三个「不」堆叠、`templates/AGENTS.md:65-70` 整块禁令。
- 框架依据：`Leading words` — Negation 会把被禁行为拉进上下文、反而提高其可用性；应给**正面目标**。
- **例外需保留**：cognition `L108`「不编造」是框架允许的硬护栏（且已配正面动作「留白并标注」），不要误删。foundation `L103`「方向单一：项目文档 → 模板」是先正面后禁令的正确范式，可作为全仓库的改写模板。

### 模式 G · no-op 与 exposition

- cognition `L12`「三种视角看同一个点」、`L32`「任何实体都可从九个维度审问」；foundation `L8`「为项目准备稳定的根级上下文文档……即拥有一致的认知基础」。
- 判据：相对**模型默认行为**是否改变行为，而非相对人类读者是否有信息量。三处整句删除，不做逐词精简。

---

## 3. 各自独有的发现（差异）

**cognition 独有**
- 收尾汇报（`L112`）是交付动作，却藏在「纪律」参考节里，第四步判据（`L89`）完全不检查它——漏做不会失败。
- 「可选补件」（`L20`：数字/反例/类比）是孤儿 reference：定义了、模板里留了槽位，但四步流程没一步提到、没一条判据检查。
- `references/domains.md` 内联名单与 SKILL.md 名单必然漂移。

**foundation 独有**
- 三份参考（`L10-47`）整块压在第一个操作（`L49`）之前，把 steps 埋在 reference 之下——框架警告这会让「attending to them 变成 coin-flip: a variance lever」。
- 「文档定位」表的「是什么」列是 `templates/*.md` 首段的 cache（`L33` 自认「完整定位见其模板首段」），属可删的缓存；同表的「解读项目时的主要来源」列是模板没有的未成文约定，应保留。
- gherkin 的 `Then`/`And` 与编号步骤、完成判据大面积重合（`L55≈L61-64`、`L56≈L67`、`L92-93≈L101-102`），应把 `Then` 收敛为只留完成保证。

---

## 4. 动作清单（按优先级）

**P0 — 两处判据（本地、便宜、收益最大，先做且只做这一批）**
1. cognition `L70` + `L104`：给「检索无果」附加举证义务（≥2 条彼此不同的检索式 + 留白去向），并取消第三步的来源豁免——无据节点只留白、不产出反转。
2. foundation `L83`：整段重写，定域「本次点名的每份文档」、统一计量单位为「章节」、要求「无变化」须写出与基准模板的逐条对照结果。照抄 `L105` 的强度。

**P1 — 指针消歧与瘦身**
3. 两处 `description` 按报告给出的替换文案重写（归还正文身份、分支收敛为一、删 no-op）。
4. foundation 立「**基准模板** / **仓库模板**」两个术语，逐处替换 `L81`/`L101`/`L105`/`L117` 的裸词「模板」；并补一句：章节若已存在于仓库模板，不计入「保留」，判为「无变化」。

**P2 — 单一事实源与顺序**
5. cognition：「反转」的两条判据收拢到「反转」节，其余处只引用；十一个领域名单只留 `domains.md` 一份；删 `L47` 对「主色调」的重复指向。
6. foundation：「不碰其他文档」只留「管辖范围」一处；授权旁路提升为「执行纪律」里的单一事实源并明确判据（「帮我初始化这四份文档」不构成旁路）；`L45` 删去「禁止依赖方向」举例。
7. foundation：把「文档定位」「模板机制」整块移到操作三之后、异常表之前。

**P3 — 措辞级**
8. 否定式改正面目标（保留 cognition `L108` 例外）；删三处 no-op 整句；foundation `L81` 用领先词「**对账**」承载「逐项核对并给处置」。

---

## 5. 共识与遗留

- **两处共识（不建议改）**：① 都不应拆分——cognition 的候选切口（第一步 vs 第二至四步）与 foundation 的三操作互斥，都不满足框架的拆分条件，且拆分要额外支付常驻 description；② 都保持 **model-invoked**——触发来自自然语言意图，用户不会说出技能名，正是 `SKILL-MECHANICS.md` 所要求的情形。
- **遗留**：本轮为静态审阅，no-op 判据在框架中明确要求「settle it by running the document, not by debate」。P0 两处判据改完后，建议以同一任务跑一轮实际对比，用行为差异而非讨论来确认。
