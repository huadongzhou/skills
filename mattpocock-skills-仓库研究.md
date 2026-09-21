# mattpocock/skills 仓库研究：多技能的管理与发布

> **研究对象**：`github.com/mattpocock/skills`（"Skills For Real Engineers"）
> **研究时间**：2026-09-20 ｜ **版本**：`v1.2.3` ｜ **规模**：169 文件 / 38 个技能
> **研究方式**：拉取仓库完整文件树 + 逐个读取关键源码与配置文件，所有结论均附文件路径可复核。

## 为什么研究它

在 skills.sh 排行榜上，`mattpocock/skills` 是**多技能单仓模式的标杆**：仓库总量 4.1M 安装，同时有 10+ 个技能各自进入全站榜单（`grill-me` 1.2M、`teach` 681K、`domain-modeling` 665K、`codebase-design` 644K、`diagnosing-bugs` 632K…）。

它的价值不在于"技能数量多"，而在于**用一整套工程化手段解决多技能仓库的固有难题**：技能生命周期如何管理、元数据如何在多个清单间保持一致、版本如何发布、跨 harness 如何兼容。本文逐项拆解。

---

## 一、仓库全貌

```
mattpocock/skills  (v1.2.3, private, MIT)
├── .agents/                    ← 决策记录与规范
│   ├── adr/                    ← 架构决策记录（0001/0002）
│   ├── install-block.md        ← 安装命令的单一事实来源
│   ├── invocation.md           ← 调用模型规范
│   └── writing-docs.md         ← 文档页写作规范
├── .changeset/                 ← changesets 版本管理（10 个未发布 changeset）
├── .claude-plugin/             ← Claude Code 插件元数据
│   ├── plugin.json             ← 插件清单（含 25 个技能的显式列表）
│   └── marketplace.json        ← 让本仓库自成一个单插件市场
├── .github/workflows/release.yml ← CI：自动开 version PR
├── .out-of-scope/              ← 明确"不做"的事（3 条）
├── docs/                       ← 人类文档页（25 篇，镜像 promoted 技能）
│   ├── engineering/            ← 18 篇
│   └── productivity/           ← 7 篇
├── scripts/
│   ├── link-skills.sh          ← 本地开发：符号链接到 harness 目录
│   ├── list-skills.sh          ← 列出全部 SKILL.md
│   └── sync-plugin-version.mjs ← 版本号双写同步（含 --check）
├── skills/                     ← 技能主体（38 个，按桶分目录）
│   ├── engineering/            ← 18 个（promoted）
│   ├── productivity/           ← 7 个（promoted）
│   ├── in-progress/            ← 9 个（beta，故意公开）
│   ├── misc/                   ← 4 个（保留但不推广）
│   └── deprecated/             ← 0 个（当前为空）
├── AGENTS.md                   ← 符号链接 → CLAUDE.md
├── CLAUDE.md                   ← 仓库约定（代理的规则文档）
├── CONTEXT.md                  ← 共享语言/术语模板
├── package.json                ← private，仅用于 changesets
└── README.md                   ← 顶层技能索引 + 理念说明
```

**关键计数**（已核验）：38 个 `SKILL.md`、38 个 `agents/openai.yaml`（1:1）、25 篇 docs 页、25 个插件清单条目。三者与"promoted 集合"完全对齐。

---

## 二、核心机制一：分桶（Bucket）管理生命周期

`CLAUDE.md` 明确定义了 5 个桶及其语义：

| 桶 | 数量 | 语义 | 进插件 | 进顶层 README | 有 docs 页 |
|---|---|---|---|---|---|
| `engineering/` | 18 | 日常代码工作 | ✅ | ✅ | ✅ |
| `productivity/` | 7 | 日常非代码工作流 | ✅ | ✅ | ✅ |
| `in-progress/` | 9 | **beta：故意公开，求反馈** | ❌ | ❌ | ❌ |
| `misc/` | 4 | 保留但极少用，不推广 | ❌ | ❌ | ❌ |
| `deprecated/` | 0 | 已弃用 | ❌ | ❌ | ❌ |

**三个值得借鉴的设计决策**：

1. **`in-progress/` 故意公开**。它不是"未完成的脏东西"，而是一个**受控的反馈通道**。`skills/in-progress/README.md` 原话："Beta. These skills are public on purpose: try them and tell me what breaks." 本地安装脚本会链接它（因为"本地安装正是反馈回路所在"），但插件和顶层 README 不收录。

2. **弃用靠删除，不靠堆积**。`skills/deprecated/README.md`："This bucket is currently empty: a retired skill is deleted, and the changeset that removes it names whatever replaced it."。意即退役即删除，并在 changeset 里写明替代品。这避免了仓库无限膨胀。

3. **推广是一个显式闸门**。桶决定一切下游行为，而不是靠人记。`CLAUDE.md` 把闸门写成硬规则：

   > "Every skill in `engineering/` or `productivity/` (the **promoted** buckets) must have a reference in the top-level `README.md` and an entry in `.claude-plugin/plugin.json`'s `skills` array... Skills in `misc/`, `in-progress/`, and `deprecated/` must not appear in either."

**对多技能仓库的启示**：技能数量增长后，真正的风险不是"装不下"，而是**"读者不知道哪些该用"**。分桶 + 推广闸门把"成熟度"变成了可机械校验的状态。

---

## 三、核心机制二：单一划分维度（Invocation）

`.agents/invocation.md` 开篇即点明：

> "Every `SKILL.md` in this repo is a skill. The one axis that splits them is **invocation**, who can reach it."

| 类型 | 可达性 | 职责 | 数量 |
|---|---|---|---|
| **User-invoked** | **仅**人类手动输入可达 | 编排（orchestrate） | 14 |
| **Model-invoked** | 人或模型均可达 | 可复用的纪律（discipline） | 11 |

（promoted 25 个中：engineering 9 user / 9 model，productivity 5 user / 2 model）

### 硬不变量

> "A user-invoked skill may invoke model-invoked skills, but it can never reach another user-invoked skill."

这条规则的价值：**user-invoked 技能是人类的入口，不能被任何东西（包括其他技能）自动触发**。否则"只在你输入时才跑"的承诺就失效了。

### 双 harness 的实现方式（同一语义，两处声明）

| 维度 | Claude Code | Codex |
|---|---|---|
| 标记 user-invoked | frontmatter `disable-model-invocation: true` | `agents/openai.yaml` → `policy.allow_implicit_invocation: false` |
| 技能选择器元数据 | — | `agents/openai.yaml` → `interface.display_name` / `interface.short_description` |

`invocation.md` 的同步纪律："Keep the two in sync: a skill is user-invoked in both harnesses or neither."

**实例对照**（`grill-me`）：

```yaml
# skills/productivity/grill-me/SKILL.md
---
name: grill-me
description: A relentless interview to sharpen a plan or design.
disable-model-invocation: true
---
```

```yaml
# skills/productivity/grill-me/agents/openai.yaml
interface:
  display_name: "Grill Me"
  short_description: "Sharpen a plan through interview"
policy:
  allow_implicit_invocation: false
```

### description 的二分写法（高价值细节）

这是本仓库最实用的一条经验，直接回答"description 该面向谁写"：

| 类型 | description 面向 | 写法 |
|---|---|---|
| **User-invoked** | **人类**（浏览斜杠命令时读） | 一句话摘要，**剥掉触发词列表**（不要 "Use when the user says…"） |
| **Model-invoked** | **模型**（用于自动触发匹配） | **保留丰富触发短语**（"Use when the user wants…, mentions…, asks for…"） |

`tdd`（model-invoked）的 description 正是后者：

```yaml
description: Test-driven development. Use when the user wants to build features or fix bugs test-first, mentions "red-green-refactor", or wants integration tests.
```

而 `grill-me`（user-invoked）只有一句人类可读的摘要，**没有触发词**，因为它不需要被模型自动匹配。

> ⚠️ 这修正了通用指南中"description 一律写成触发规则"的简化说法：**触发短语只对 model-invoked 技能有意义**。

---

## 四、核心机制三：三清单同步（多技能仓库的最大维护负担）

promoted 技能的元数据分散在 **3 个地方**，必须保持一致：

| # | 位置 | 作用 | 数量 |
|---|---|---|---|
| 1 | 顶层 `README.md` 的 Reference 章节 | 人类浏览入口 | 25 |
| 2 | `.claude-plugin/plugin.json` 的 `skills[]` | 插件实际装载清单 | 25 |
| 3 | `docs/<bucket>/<name>.md` | 人类文档页（发布到 aihero.dev/skills-<name>） | 25 |

`CLAUDE.md` 用三条硬规则把它们钉死：

1. promoted 技能**必须**同时出现在 ① 与 ②；非 promoted **必须不出现**。
2. 顶层 README 中每个技能名**必须**链接到其 `SKILL.md`。
3. 每个桶的 `README.md` 列出桶内全部技能 + 一句话描述 + 链接；promoted 桶的 README 与顶层 README 需按 **User-invoked / Model-invoked** 分组，非 promoted 桶用**扁平列表**。

另有一条"路由器同步"规则：

> `ask-matt` 是映射所有 user-reachable 技能的 router。"whenever you add, rename, remove, or change how a user-reachable skill fits the flows, re-read `ask-matt`'s `SKILL.md` and update it... a new skill it never mentions, or a stale one it still routes to, is a router that lies."

**注意**：这三处同步**没有自动化脚本**，靠 `CLAUDE.md` 的约定 + 人工纪律维持。这是该仓库有意为之的取舍：把"该做什么"写进代理的规则文档，让代理在改动时自己遵守。

---

## 五、核心机制四：发布工程化（Changesets + CI）

### 版本管理：changesets，但不发 npm

`package.json`：

```json
{
  "name": "mattpocock-skills",
  "version": "1.2.3",
  "private": true,
  "scripts": {
    "version": "changeset version && node scripts/sync-plugin-version.mjs",
    "check-plugin-version": "node scripts/sync-plugin-version.mjs --check"
  },
  "devDependencies": {
    "@changesets/changelog-github": "^0.7.0",
    "@changesets/cli": "^2.30.0"
  }
}
```

`.changeset/config.json` 关键项：

```json
{
  "changelog": ["@changesets/changelog-github", { "repo": "mattpocock/skills" }],
  "commit": false,
  "privatePackages": { "version": true, "tag": true },
  "baseBranch": "main"
}
```

**要点**：包是 `private` 的（不发 npm），但 `privatePackages.version/tag` 设为 `true`，**借用 changesets 的版本流来管理"技能集合"的版本**。这是一个很巧妙的用法：技能不是包，但仍需要版本号与 CHANGELOG。

### CI：push 到 main 即开 version PR

`.github/workflows/release.yml`：

```yaml
on:
  push:
    branches: [main]
jobs:
  release:
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with: { node-version: 22 }
      - run: npm ci
      - uses: changesets/action@v1
        with:
          version: npm run version
          publish: npx changeset tag
          commit: "chore: version skills"
          title: "chore: version skills"
        env:
          GITHUB_TOKEN: ${{ secrets.GITHUB_TOKEN }}
```

流程：**合入带 changeset 的 PR → CI 自动开一个 version PR（含 CHANGELOG + 版本号提升）→ 合入该 PR 即打 tag**。作者从不需要手动改版本号。

### 版本号双写问题与解法

版本号存在两处，必须一致：

- `package.json` → `version`
- `.claude-plugin/plugin.json` → `version`

`scripts/sync-plugin-version.mjs` 解决它：从 `package.json` 读版本，**只替换 `plugin.json` 中的 version 行**（保留键顺序与格式），`--check` 模式下不改文件、不一致则退出码 1。

```js
// 只重写 version 行，保留键顺序与格式
const updated = source.replace(/("version"\s*:\s*")[^"]*(")/, `$1${version}$2`);
```

**这解决了一个真实事故**：仓库曾出现 `plugin.json` 被手动升到 1.2.1、而 `package.json` 停在 1.2.0 的漂移。现在 `npm run version` 保证同一次 version PR 携带两个文件；`npm run check-plugin-version` 可在 CI 中检测漂移。

> **通用教训**：任何"同一事实存两处"的设计都必须配一个同步脚本 + 一个 `--check` 模式。多技能仓库里这类双写点很多（README / plugin.json / docs / openai.yaml）。

---

## 六、核心机制五：双轨分发（互斥）

这是该仓库在"发布"上最清晰的产品决策。

| 轨道 | 命令 | 性质 | 更新方式 |
|---|---|---|---|
| **Claude Code 插件** | `claude plugins install mattpocock-skills` | 受管、**只读**的整包 | **自动更新**（在官方市场 `claude-plugins-official` 中） |
| **skills.sh** | `npx skills@latest add mattpocock/skills` | 写入**你可编辑**的普通文件 | 需手动 `npx skills update` |

`.agents/install-block.md` 明确了互斥性：

> "The two routes are exclusive... Installing both leaves the user with every skill twice: always say 'pick one'."

同时解释了 `.claude-plugin/marketplace.json` 的定位：它让仓库自成一个单插件市场（`/plugin marketplace add mattpocock/skills`），但**官方市场收录后已被取代**，仅作为"安装未发布 commit 或 fork"的 fallback，且**不向用户文档化**。

### 安装命令的单一事实来源

`.agents/install-block.md` 是安装文案的 canonical source：

> "One install story, one wording. `README.md`, `.changeset/*`, and every page under `docs/` must say **this** and nothing else. Change it here first, then propagate."

它用 `<canonical-block name="...">` 标记了三个变体：`claude-code`、`skills-sh-whole-set`、`skills-sh-one-skill`。并规定 `skills@latest` 是三处统一拼写。

**注意**：`docs/` 页面**不是**这个 block 的消费者。站点会在正文上方渲染安装组件，页面里再写一遍就是重复，所以那些块被**删除而非修正**。

> **通用教训**：当安装命令出现在 N 个地方时，先在 N 之外建立"唯一来源"，再让所有地方引用它。这个仓库甚至为此写了专门的规范文档。

---

## 七、核心机制六：本地开发链接

`scripts/link-skills.sh`（**明确声明为 dev-only，不对外支持**）把仓库内技能符号链接到两个 harness 目录：

```
~/.claude/skills    ← Claude Code
~/.agents/skills    ← Codex 及其他 Agent Skills 兼容 harness
```

关键行为：

- 用 `find` 收集所有 `SKILL.md`，**排除 `deprecated/` 与 `misc/`**，**保留 `in-progress/`**（理由见第二节）。
- 每个条目是指向仓库的**符号链接**，因此"a `git pull` is all that's needed to keep installed skills up to date"。
- 有**自污染防护**：若目标目录本身是指向本仓库的符号链接，会检测并报错退出，避免把 per-skill 链接写回仓库自身。

```bash
# 自污染防护的核心逻辑
if [ -L "$DEST" ]; then
  resolved="$(readlink -f "$DEST")"
  case "$resolved" in
    "$REPO"|"$REPO"/*) echo "error: $DEST is a symlink into this repo" >&2; exit 1 ;;
  esac
fi
```

> **通用教训**：多技能仓库的开发者需要"改一处、全部 harness 立即生效"的能力。符号链接 + `git pull` 是零成本方案；但必须防住"链接回自身"这个坑。

---

## 八、核心机制七：跨 harness 一致性与文档纪律

### AGENTS.md = CLAUDE.md 的符号链接

```
AGENTS.md  (mode 120000 → CLAUDE.md)
CLAUDE.md  (mode 100644, 真实文件)
```

用**一个符号链接**让 Codex 与 Claude Code 读取完全相同的规则，避免"两份规则各自漂移"。这是零成本的跨 harness 一致性方案。

### 文档页四段式

promoted 技能各有一篇 `docs/<bucket>/<name>.md`，固定四段：

1. **What it does**
2. **When to reach for it**
3. **Common questions**
4. **It's working if**

发布 URL 为 `https://aihero.dev/skills-<skill-name>`（**与桶无关**，`docs/` 的分目录仅为仓库组织）。

### 写作禁令：禁止 em-dash

`CLAUDE.md` 有一条罕见但值得注意的规则：

> "No em-dashes anywhere in this repo's prose... Where a sentence reaches for one, rewrite it instead with a comma, colon, period, parentheses, or a conjunction, whichever the sentence actually wants; **never do a blind character substitution**."

仓库曾专门做一次全仓清理（`changeset/remove-em-dashes-repo-wide.md`），要求**手写改写**而非机械替换。

### 其他约定

- 技能目录名：**kebab-case**（`grill-me`、`improve-codebase-architecture`）
- 调用形式：斜杠命令 `/grill-me`
- 元数据风格：`short_description` 为**简短动词短语，结尾不加句号**
- 提交信息：Conventional Commits（`feat:`、`chore:`）
- 决策留痕：`.agents/adr/`（架构决策）、`.out-of-scope/`（明确不做的事，3 条）

---

## 九、技能间依赖的表达方式

`.agents/invocation.md` 规定：依赖必须写成**显式的 Skill 工具调用指令**：

> 写成 `Call the Skill tool with "grilling"`，而**不是** `../other-skill/FILE.md` 跨目录引用，也**不是**留给模型自行解释的裸 `/skill` 提及。

理由："Naming the tool is what gets it fired"：多数 harness 把技能调用暴露为一个工具，明确点名命中率远高于在散文里丢一个 `/name`。

配套细节：

- **去掉前导 `/`**，使表述保持 harness 中立。
- **Skill 工具一次只接一个技能**。需要两个技能就写两次调用："Call the Skill tool twice, for 'grilling' and 'domain-modeling'"。
- 共享参考文档**放在拥有它的技能内部**，其他技能通过调用该技能来触达，而非跨目录链接。
- 该约定**仅对 model-invoked 技能成立**。若前置条件是 user-invoked 技能（如 `setup-matt-pocock-skills`），必须改写成给人类的指令："tell the user to run `/setup-matt-pocock-skills`"。
- **被动 vs 主动**：仅仅"读 `CONTEXT.md` 取词"只是一句散文指针，**不算**调用 `domain-modeling` 技能；只有主动的构建/锐化纪律才算。

---

## 十、值得注意的"缺失"：没有 skills.sh.json

仓库**没有** `skills.sh.json`（已核验文件树）。它的 skills.sh 可发现性依赖：

1. `skills/<bucket>/<name>/SKILL.md` 的标准目录结构；
2. `.claude-plugin/plugin.json` 的显式 `skills[]` 数组（走 CLI 的**插件清单发现**路径）。

**含义**：`skills.sh.json` 是**可选**的仓库页分组配置，不是发布必需品。仓库页的分组可以靠目录结构 + 插件清单自然形成。

---

## 十一、可迁移的模式清单

按"移植成本 / 收益"排序，供本仓库参考：

| 优先级 | 模式 | 移植成本 | 收益 |
|---|---|---|---|
| **P0** | 分桶 + 推广闸门（promoted 集合显式化） | 低 | 技能增长后仍可读、可校验 |
| **P0** | `CLAUDE.md`/`AGENTS.md` 写清同步规则，让代理自遵守 | 低 | 免去三清单同步的人工遗漏 |
| **P0** | description 按 invocation 二分（人类摘要 vs 模型触发词） | 极低 | 直接改善触发准确率与可读性 |
| **P1** | changesets + CI 自动 version PR | 中 | 版本/CHANGELOG 自动化，零手工 |
| **P1** | 双写点同步脚本 + `--check` | 中 | 消除版本漂移类事故 |
| **P1** | `link-skills.sh` 符号链接本地开发流 | 低 | 改一处全 harness 生效 |
| **P2** | `AGENTS.md` → `CLAUDE.md` 符号链接 | 极低 | 跨 harness 规则零漂移 |
| **P2** | 安装命令 canonical block | 低 | 多处以安装文案不再打架 |
| **P2** | docs 页四段式模板 | 中 | 文档质量下限有保障 |
| **P2** | ADR / out-of-scope 留痕 | 低 | 决策可追溯，减少反复讨论 |
| **P3** | Claude Code 插件双轨分发 | 高 | 面向 Claude Code 用户的最佳体验 |

---

## 十二、与本仓库的对照

本仓库 `D:\project\skills`（`cognition` + `foundation`）目前处于 mattpocock 模型的**最早期阶段**：

| 维度 | mattpocock/skills | 本仓库现状 | 差距 |
|---|---|---|---|
| 技能数 | 38 | 2 | 尚不需要分桶 |
| 目录 | `skills/<bucket>/<name>/` | `skills/<name>/` | 可先保持扁平（CLI 支持） |
| 调用模型 | 明确 user/model 二分 | 未声明 | **建议优先补**：为两个技能标注 invocation |
| 插件清单 | 有 `plugin.json` | 无 | 视是否需要 Claude Code 插件 |
| 版本管理 | changesets + CI | 无 | 技能稳定后再引入 |
| 文档页 | `docs/` 25 篇 | 无 | 可选 |
| 本地链接 | `link-skills.sh` | 无 | 需要多 harness 调试时再加 |
| 仓库页配置 | 无 `skills.sh.json` | 无 | 一致 |

**建议的下一步（按收益排序）**：

1. **先补 invocation 语义**：判断 `cognition`/`foundation` 是 user-invoked 还是 model-invoked，据此重写 `description`（这是唯一"现在做就有收益"的一项）。
2. **清理 `skills/foundation-workspace/`**：eval 产物不应混入技能容器（详见发布指南）。
3. **暂不引入 changesets/插件**：2 个技能引入 CI 属于过度工程；等技能数量与迭代频率上来再补。
4. **技能数超过 5 个时**，再引入分桶 + 推广闸门 + 三清单同步规则。

---

## 附：关键文件索引（可复核）

| 文件 | 揭示的机制 |
|---|---|
| `CLAUDE.md` | 分桶规则、三清单同步硬约束、em-dash 禁令 |
| `.agents/invocation.md` | user/model-invoked 定义、不变量、依赖表达约定 |
| `.agents/install-block.md` | 安装命令 canonical block、双轨互斥性 |
| `.agents/writing-docs.md` | docs 页四段式模板 |
| `.agents/adr/0002-ship-as-a-claude-code-plugin.md` | 为何先做 Claude 插件 |
| `.claude-plugin/plugin.json` | 25 个 promoted 技能的显式清单 |
| `.claude-plugin/marketplace.json` | 单插件市场（fallback） |
| `package.json` + `.changeset/config.json` | private 包借 changesets 管版本 |
| `scripts/sync-plugin-version.mjs` | 版本双写同步 + `--check` |
| `scripts/link-skills.sh` | 符号链接本地开发 + 自污染防护 |
| `.github/workflows/release.yml` | 自动 version PR |
| `skills/*/README.md` | 桶级技能清单（含 User/Model 分组） |
| `skills/*/*/agents/openai.yaml` | Codex 元数据（display_name / policy） |
