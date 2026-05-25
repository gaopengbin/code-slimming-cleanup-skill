# 代码瘦身与清理 Skill

一个可复用的 Claude Code skill，用于安全地让代码更少、更简单、更干净，同时避免把清理任务变成失控的大重构。

## 它适合做什么

- 在不改变行为的前提下减少代码量
- 删除已证明无用的死代码
- 清理未使用的 import、变量、辅助函数和分支
- 简化不必要的抽象、间接层和配置
- 在行为确实相同时合并重复逻辑
- 避免高风险的顺手重构

## 作为 Claude Code 插件安装

在 Claude Code 中，可以按你的 Claude Code 插件流程添加或安装这个仓库。

skill 文件位置：

```text
skills/code-slimming-cleanup/SKILL.md
```

## 核心理念

好的清理应该让 reviewer 相信：现在更少的代码完成了同样的事情。

这个 skill 有意保持保守：

- 只删除任务范围内的代码。
- 删除前先证明代码确实无用。
- 优先删除，而不是引入新抽象。
- 除非用户明确要求，否则保持行为不变。
- 最后说明删除了什么，以及如何验证。

## 仓库结构

```text
.
├── README.md
├── README.zh.md
├── .claude-plugin/
│   └── plugin.json
└── skills/
    └── code-slimming-cleanup/
        └── SKILL.md
```

## 许可

MIT
