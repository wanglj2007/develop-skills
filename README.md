# develop-skills

面向日常软件开发的可复用技能库。每个技能以独立目录维护，通过 `SKILL.md` 说明适用场景、约定和实施步骤，供 AI 编程助手或开发者在对应任务中引用。

## 目录结构

```text
develop-skills/
├── csharp-api/
│   └── SKILL.md
└── README.md
```

## 技能列表

| 技能 | 适用场景 |
| --- | --- |
| [csharp-api](./csharp-api/SKILL.md) | 在 ASP.NET Core 等 C# 后端项目中新增、重构或审查业务管理 API；覆盖 Controller、Service 和 Models 分层，以及列表、保存、启用、禁用与删除接口约定。 |

## 使用方式

进入具体技能目录，阅读其 `SKILL.md`，并在执行任务时遵循其中的约定。例如：

```bash
cat csharp-api/SKILL.md
```

## 贡献新技能

1. 在仓库根目录创建语义清晰的技能目录。
2. 在目录内新增 `SKILL.md`，包含 YAML 元数据（至少含 `name` 与 `description`）以及可执行的使用说明。
3. 在本 README 的“技能列表”中补充该技能的链接和适用场景。
4. 确保技能内容聚焦、可复用，并与现有约定保持一致。
