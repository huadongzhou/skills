# cognition 技能审阅报告 — writing-for-agents 框架

- 审阅对象：`D:\project\skills\skills\cognition\`（SKILL.md / references/dimensions.md / references/domains.md / templates/knowledge-doc.md）
- 框架：Matt Pocock `writing-for-agents`（SKILL.md + SKILL-MECHANICS.md）
- 纪律：只读审阅，未改动 `skills\` 下任何文件。

---

## 1. 一句话结论

**防赶工的全部希望被押在两处 `🔴 CHECKPOINT · 🛑 STOP` 上，而它们是内联的用户确认、不清空上下文，构不成框架所说的「真实上下文边界」；真正能抵挡赶工的完成判据反而在第二步留了一个无限逃逸口（「或一条明确的检索无果」），又被兜底原则豁免到第三步——于是四步流程里没有任何一处真正抵抗早停。**

---

## 2. 关键发现（按严重度）

### 高

**H1 · 两处 STOP 不是上下文边界，且反向增加了赶工拉力**
- 位置：`SKILL.md:59`、`SKILL.md:85`
- 问题：L59「骨架定稿……先交用户确认，确认后再进入第二步」、L85 同构。二者都在同一次内联运行内，第二至四步始终留在上下文里，STOP 一个 token 也没清掉。
- 框架依据：`Steps and completion criteria` —「Hiding only works across a real context boundary (a hand-off or a subagent dispatch; an inline call leaves the later steps in context and clears nothing)」。更糟的是同一节指出 **post-completion steps 是拉力的来源**：CHECKPOINT 把「骨架完成」变成一个用户可见里程碑，等于给第一步额外加了一次拉力，却没有加任何抵抗力。
- 建议改法：
  1. 把 L59 整行替换为：
     > **🔴 用户审批门 · 骨架**：把骨架（领域判定、九维主次与并入、各维待答问题）交用户确认后再进第二步；用户已明确要求直接产出成品（如「直接生成并保存」）时跳过，改为交付时一并汇报。
     （删掉 `🛑 STOP`，只保留它真实的功能：人类审批。）
  2. 抵抗力补在判据上（见 H2、M3 的改法），不靠隐藏步骤。
  3. **不要为此拆分技能**：框架规定先「sharpen the bound first」，只有判据「irreducibly fuzzy *and* you observe the rush」才拆分，而拆分要花一份常驻上下文负载。

**H2 · 第二步判据的「检索无果」是无限逃逸口，并被自动豁免到第三步**
- 位置：`SKILL.md:70`、`SKILL.md:67`、`SKILL.md:97-101`、`SKILL.md:104`
- 问题：L70「每个待答问题都有至少一条可追溯来源，**或一条明确的「检索无果」**」——该析取项无任何约束（L67 只说「检索不到就写」，L97 只约束工具不可用一种情形）。更严重的是 L104「某节点标注「检索无果」时，第三步的来源要求对该节点自动豁免」，使一个节点可先标无果、再写**无来源的反转**，全篇的 exhaustiveness bar 由此塌陷。
- 框架依据：`Steps and completion criteria` —「**Demand**: how much it requires… "every rule applied" binds a body of flat reference just as "every step done" binds a sequence」；判据要「both checkable and exhaustive」。当前判据 checkable 但可被单一 token 满足，demand 为零。
- 建议改法：
  - L70 替换为：
    > **完成判据**：每个待答问题都有至少一条可追溯来源；标为「检索无果」的问题，须附已尝试的检索式（≥2 条且彼此不同）与该问题的留白去向。
  - L104 替换为：
    > **兜底原则**：留白优先于推测。某节点标「检索无果」时，第三步的要求降为：该节点只写「检索无果：<缺什么>」，**不产出反转**；反转留给有来源的节点。

**H3 · description 缓存了正文已经承载的身份信息**
- 位置：`SKILL.md:3`
- 问题：「产出九维逐层、从表层默认认知挖到深层真相的知识文档，每个节点交齐黑话/白话/案例三件套与有据的反转」——「九维」由 L30-44 定义、「三件套」由 L10-18 定义、「反转」由 L22-28 定义。指针里这三样全是正文的复述。同句触发段「当用户要求深挖某个实体，**或要求构建/生成知识深度树文档**时使用」是**一个分支写了两遍**：两条都是「产出一份知识深度树」，走的是同一条四步路径，不是两个分支。
- 框架依据：`Context pointers` —「**Cut identity the body already carries**」「**One trigger per branch**：Synonyms that rename a single branch are one branch written twice」；「Every word of an always-loaded pointer costs on every turn」。
- 建议改法：L3 整行替换为
  > description: 知识深度树：给定任意实体，产出一份把默认认知翻转过来、且每处反转都有据的知识文档。当用户要求深挖某个实体时使用。

  领先词「知识深度树」仍居首，正文身份全部归还正文，分支收敛为一。

### 中

**M1 · 「反转」的规则碎在五处，co-location 不成立**
- 位置：`SKILL.md:24-28`（定义 + 外行反应判据 + 每节点至少一条 + 主色调指针）、`SKILL.md:76-78`（步骤用法）、`SKILL.md:81`（又一条「判据」）、`SKILL.md:98`（来源矛盾时的例外）、`references/domains.md:50-66`（主色调表）
- 问题：同一定义、质量闸门、数量要求、步骤用法、例外分散在四节；L26 与 L81 都叫「判据」，一个是质量闸门、一个是完整性闸门，读一处带不出另一处。
- 框架依据：`Information hierarchy` —「**Co-location** … Keep a concept's definition, rules, and caveats under one heading rather than scattered, so reading one part brings its neighbours with it.」
- 建议改法：把完整性要求并入 L28，L28 改为「每个保留维度至少一条反转；每条反转须指出它推翻的具体默认认知并附至少一个支撑来源。全篇反转按树序排列……」。L81 改为「每个保留维度三件套齐全，反转按「反转」节两条判据齐备。」——一处定义，两处只引用。

**M2 · 十一个领域名单有两个家（单一事实源被破坏）**
- 位置：`SKILL.md:53`（内联十一项名单）vs `references/domains.md:7-19`（同名名单，另带代号、典型实体、归属读法）
- 问题：增删一个领域要改两个文件；SKILL.md 那份还是残缺版（无代号），两份必然漂移。
- 框架依据：`Pruning` —「Keep each meaning in a **single source of truth**: one authoritative place, so changing the behaviour is a one-place edit.」
- 建议改法：L53 替换为
  > 1. 判定领域（十一个，可多标；名单、代号与判定次序见 [`references/domains.md`](references/domains.md)）；

**M3 · 收尾汇报藏在「纪律」里，不在任何完成判据内**
- 位置：`SKILL.md:112`
- 问题：「收尾汇报：领域判定、维度主次与并入、各节点反转数、检索无果项、时效标注」是**交付动作**，却放在 L106 的「纪律」（reference）节，且第四步判据 L89 完全不检查它——漏做不会失败。
- 框架依据：`Information hierarchy`（steps 与 reference 不混放）+ `Steps and completion criteria`（判据要可判定）。
- 建议改法：删除 L112，把 L89 判据末尾改为「……；树序完整无断档；交付时一并汇报：领域判定、维度主次与并入、各节点反转数、检索无果项、时效标注。」

**M4 · 「可选补件」是孤儿 reference**
- 位置：`SKILL.md:20`（定义 数字/反例/类比）、`templates/knowledge-doc.md:26-28`（槽位）
- 问题：定义在正文、槽位在模板，但四步里没一步提到它、没一条判据检查它，「按节点需要加」也无触发条件。
- 框架依据：`Pruning` —「Check every line for **relevance**: does it still bear on what the document does? A line loses relevance by never bearing on the task」。
- 建议改法：在第三步 L79 后加一条 bullet：
  > - 只在节点确实需要时补 数字 / 反例 / 类比（见「三件套」节的可选补件）。

**M5 · 两个 in-file 指针是内容清单，未编码触发条件**
- 位置：`SKILL.md:46-47`
- 问题：两行都在罗列「里面有什么」（查什么、反转藏哪、覆盖矩阵、主色调），而**何时去读**写在别处（L54 定主次、L63 逐维检索、L78 取主色调）。指针与触发分离。
- 框架依据：`Context pointers` —「A **context pointer** … names some out-of-context material and **encodes the condition for reaching it**」。
- 建议改法：
  > 写某一维度的填法（查什么、反转藏哪、黑话样本、好案例）时读 [`references/dimensions.md`](references/dimensions.md)。
  > 判定领域、定九维主次与并入、取反转主色调时读 [`references/domains.md`](references/domains.md)。

### 低

**L1 · 靠否定表达的四处，可改正面目标**
- `SKILL.md:108`「**不编造**：检索无果处留白并标注……」→「**有据才写**：检索无果处留白并标注……」
- `SKILL.md:110`「……先查覆盖矩阵，**不为单个实体新造维度**」→「……先查覆盖矩阵，用现有九维落位」（正面目标已足够，否定可删）
- `SKILL.md:79`「用真实事件**而非虚构示例**」→「案例真实可追溯：谁、何时、发生了什么，全部可查证」
- 框架依据：`Leading words` —「**Negation** is the failure mode beside this lever: steering by prohibition drags the forbidden behaviour into context and makes it *more* available」。

**L2 · 两处 no-op 残留**
- `SKILL.md:12`「三种视角看同一个点」——不改变任何行为，下面的表已经说完；整句删。
- `SKILL.md:32`「任何实体都可从九个维度审问」——同属说明性铺垫，可并入「九维对全领域通用」。
- 框架依据：`Pruning` —「Hunt **no-ops** sentence by sentence… When a sentence fails, delete the whole sentence rather than trim words from it.」

**L3 · 指向 domains.md 的同一内容被指了两次**
- `SKILL.md:28` 与 `SKILL.md:47` 都指向 domains.md 的「反转主色调」。保留 L28（与「反转」概念同处，co-location 更好），L47 删去该项。

---

## 3. 通过项

- **description 前置领先词**：L3 首词即「知识深度树」，指针的触发工作放在了最前（`Context pointers`）。
- **反转的二元闸门**：L26「单独拿给一个外行看，他的反应是「等等，真的？」……若反应是「嗯，有道理」……删掉」——正是框架 `Leading words` 里 `red` 式的可观测二元状态，比任何形容词都硬。
- **步骤名本身就是领先词**：立树 / 检索取证 / 逐节点落笔 / 成文交付（L51、61、72、83），四个 token 各自锚定一段行为。
- **披露位置正确**：`dimensions.md`、`domains.md`、`knowledge-doc.md` 全在指针之后，逃出常驻负载；正文里的九维表每个分支都要用，内联是对的（`Information hierarchy` 的 disclosure 测试）。
- **第三步判据既 checkable 又 exhaustive**：L81「**每个**保留维度三件套齐全……**每条**反转都指出它推翻的具体默认认知、附至少一个支撑来源」。
- **异常与兜底表是分支式 reference**：六行触发条件各对应不同路径，不是 no-op 复述。
- **第二步的来源纪律有真拉力**：L66「反转这类反直觉结论，至少两个独立来源」——针对性护栏，非泛泛的「要严谨」。

---

## 4. 不建议改的

- **两处 CHECKPOINT 本身保留**。作为**人类审批门**它们是对的：`The two loads` 明说认知负载「is the price of human agency; spend it where human judgement matters」——骨架定稿与检索无果项正是人类判断该介入处。要改的只是标签与预期，不是它们的存在。
- **不按序列拆分**。唯一候选切口是「第一步 vs 第二至四步」，但内联运行没有真实边界，拆出去只多花一份常驻 description 而不隐藏任何东西。
- **保持 model-invoked**。触发词「深挖某个实体」是自然语言意图，用户未必知道技能名；集合内也没有 router 或其它技能要调用它——正是 `SKILL-MECHANICS.md`「Pick model-invocation only when the agent must reach the skill on its own」成立的情形。若日后确认用户总是手敲技能名，再翻成 user-invoked，常驻负载归零。
- **`references/` 分成两份是对的**。二者是不同查找时机（步骤一查领域、步骤三查维度填法），各自成篇，符合「push behind a pointer what only some branches reach」。
- **`templates/knowledge-doc.md:34` 的 HTML 注释保留**。它看似「给作者看的话会污染成品」，实为 agent 成文时刻的指令载体，且第四步判据 L89 明确要求删除它——配对关系，不是重复。
- **L108「不编造」保留否定形式**。`Leading words` 给否定留了例外：「A prohibition earns its place only as a hard guardrail you cannot phrase positively; even then, pair it with the positive target」——编造是最高代价的失败模式，且已配正面动作「留白并标注」。
- **`references/dimensions.md` 的七项固定结构保留**。它是扁平 peer-set reference 的正当形态（`Information hierarchy`：legitimately flat peer-set, a fine arrangement, not a smell），九维同构才便于对照。
