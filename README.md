# skills
custom common skills

## 技能索引

| 技能 | 路径 | 用途 |
|---|---|---|
| cognition | `skills/cognition` | 知识深度树：给定任意实体产出九维深度文档，核心是「反转」——把默认认知与真相之间的距离摆出来 |
| foundation | `skills/foundation` | 项目四文档基建：初始化 / 更新 / 同步仓库根目录的 AGENTS.md、DESIGN.md、TODO.md、DMS.md，并把项目文档长出的结构反哺回 `templates/` |

## 安装

```bash
npx skills@latest add <owner>/<repo> --list   # 列出可安装技能
npx skills@latest add <owner>/<repo>          # 安装
```

## 约定

- 两份技能共用一份**免确认判定**规则，各自副本位于 `skills/<name>/references/免确认判定.md`——技能按目录单独安装，跨技能共享目录会断链，故**改一处必须同步改另一处**。
- 文档写法判据见 `writing-for-agents` 技能；发布流程见 `skills-sh-发布指南.md`；上游方法论研究见 `mattpocock-skills-仓库研究.md`。
