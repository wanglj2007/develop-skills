---
name: csharp-api
description: 在任意基于 C# 的后端项目中新增、重构或审查业务管理 API 时使用。识别到 C# 接口开发需求时，优先询问用户是否按本技能规范实施；用户提及“c#-api”时，将其视为本技能 csharp-api 的简写触发词并直接执行。适用于 ASP.NET Core 的 Controller、Service 与 Models 分层设计和修改，尤其是涉及列表、保存、新增、编辑、启用、禁用、删除或批量删除接口时；统一接口路由、HTTP 方法、模型目录和服务分层，并以项目现有约定为准。
---

# C# 后端 API 开发规范

## 简写触发词

用户使用 `c#-api` 时，按本技能执行。例如：`c#-api：实现角色管理接口`。

## 开始前确认

当用户提出 C# 接口开发、修改或重构需求，但没有提及 `csharp-api` 或 `c#-api` 时，先询问：“是否按 csharp-api SKILL 规范开发？”得到确认后再按本技能实施。用户明确使用任一触发词时，无需重复询问，直接执行。

在任意 C# 后端项目开发业务管理 API 时，先阅读仓库及子目录的项目约定（例如 `AGENTS.md`、贡献指南），再检查相邻模块的实现。特殊业务要求和当前项目既有约定优先于本规范；未被覆盖的部分按本规范实现。

## 设计原则

- API 只使用 `GET` 和 `POST`。不要新增 `PUT`、`PATCH` 或 `DELETE` 接口。
- 一个业务域使用完整分层，避免 Controller 直接访问 DbContext 或承载业务逻辑。
- 请求、查询和响应模型不定义在 Controller 或 Service 接口文件中，统一放入 `Models` 的对应业务目录。
- 接口返回、鉴权、错误码和分页格式遵循项目已有的响应包装、控制器基类及相邻模块约定。
- 先复用既有服务、实体和通用模型；不要因本规范创建重复抽象。

## 目录与命名

以业务域 `Role` 为例：

```text
YourProject/
├── Controllers/
│   └── RolesController.cs
├── Services/
│   └── Roles/
│       ├── IRoleService.cs
│       └── RoleService.cs
└── Models/
    └── Roles/
        └── RoleModels.cs
```

遵循以下规则：

- Controller 使用复数业务名：`RolesController`、`UsersController`。
- 服务目录使用复数业务名：`Services/Roles/`；服务接口和实现使用单数：`IRoleService`、`RoleService`。
- 模型目录与业务域对应：`Models/Roles/`。模型文件使用清晰的业务名称，例如 `RoleModels.cs`；当模型较多时可按用途拆成 `RoleRequests.cs`、`RoleResponses.cs`、`RoleQueries.cs`。
- Controller、Service 接口、Service 实现都显式引用对应的 `YourProject.Models.XXs` 命名空间。
- 公开请求、查询、列表项和响应模型使用描述性名称，如 `SaveRoleRequest`、`RoleListQuery`、`RoleListItem`、`RoleListResult`。
- C# XML 文档注释使用多行 `summary` 格式。

## 路由与 HTTP 方法

控制器路由以资源名为根。列表、保存、状态切换和删除使用固定动作路由：

| 场景 | HTTP 方法 | 路由 | 请求体 / 参数 |
| --- | --- | --- | --- |
| 查询列表 | `GET` | `/xxs/list` | 查询参数；分页字段使用项目统一命名 |
| 新增或更新 | `POST` | `/xxs/save` | `SaveXXRequest`，通过主键 `Id` 区分 |
| 启用 | `POST` | `/xxs/{id}/enable` | 无请求体，除非业务确有额外参数 |
| 禁用 | `POST` | `/xxs/{id}/disable` | 无请求体，除非业务确有额外参数 |
| 单个或批量删除 | `POST` | `/xxs/delete` | 请求体是 ID 数组 |

删除接口必须复用同一个 `/delete` 接口。单删时前端传单元素数组，批量删除传多个 ID；不要分别创建 `/{id}`、`/batch-delete` 等接口。除非有明确的业务要求，删除请求体直接使用 `IReadOnlyList<string>` 或 `List<string>`，保持 JSON 结构为数组。

示例：

```csharp
[HttpGet("list")]
public async Task<IActionResult> List([FromQuery] RoleListQuery query, CancellationToken ct) =>
    Ok(await roleService.GetListAsync(query, ct));

[HttpPost("save")]
public async Task<IActionResult> Save([FromBody] SaveRoleRequest request, CancellationToken ct) =>
    Ok(await roleService.SaveAsync(request, User.UserId(), ct));

[HttpPost("{id}/enable")]
public async Task<IActionResult> Enable(string id, CancellationToken ct) =>
    Ok(await roleService.EnableAsync(id, User.UserId(), ct));

[HttpPost("{id}/disable")]
public async Task<IActionResult> Disable(string id, CancellationToken ct) =>
    Ok(await roleService.DisableAsync(id, User.UserId(), ct));

[HttpPost("delete")]
public async Task<IActionResult> Delete([FromBody] IReadOnlyList<string> ids, CancellationToken ct) =>
    Ok(await roleService.DeleteAsync(ids, User.UserId(), ct));
```

## 保存规则

- 新增和编辑界面都调用 `/save`，不要分别暴露 `/create`、`/update` 或 `/{id}` 更新接口。
- `SaveXXRequest.Id` 为空或空白时新增；有值时更新指定记录。
- 新增时校验必填字段和唯一约束；更新时先查询未删除的记录，再校验允许修改的字段。
- 更新不可修改的业务标识（例如编码）时，在模型或服务中明确限制；不要依赖前端禁用输入框作为唯一保障。
- 写入创建人、更新人和时间字段时遵循实体既有审计字段约定。

## 状态与删除规则

- `EnableAsync` 与 `DisableAsync` 是两个独立服务方法和两个独立 API，不使用通用状态接口替代。
- 对系统内置记录、受引用记录或其他受保护对象，服务层必须校验并返回明确业务错误，不能只在前端隐藏按钮。
- 删除前过滤空 ID 并去重；ID 数组为空时返回“请选择…”类业务错误。
- 删除应遵循实体既有软删除约定。删除主对象时，如业务关系要求同步清理关联记录（例如角色与用户关系），在同一保存操作中处理。
- 批量操作应一次性查询和处理，避免逐条保存导致部分成功和额外数据库往返。

## 实施步骤

1. 确认业务域名称、实体、授权策略以及是否存在特殊业务要求。
2. 在 `Models/XXs` 定义或补充请求、查询和响应模型，并添加数据校验特性。
3. 在 `Services/XXs/IXXService.cs` 声明列表、保存、启用、禁用、删除等服务契约。
4. 在 `Services/XXs/XXService.cs` 实现校验、查询、审计字段和事务内关联数据处理。
5. 在 `Controllers/XXController.cs` 仅进行路由、模型绑定、当前用户传递和结果包装；不要放入业务逻辑。
6. 同步更新前端 API 调用：列表使用 `/list`，保存使用 `/save`，删除传 ID 数组到 `/delete`。
7. 搜索旧的 `PUT`、`PATCH`、`DELETE`、`create`、`update`、单独批量删除等路由，确认本次业务域没有保留冲突接口。
8. 执行受影响项目构建；涉及控制台调用时同时执行类型检查，并核对 OpenAPI 或关键请求。

## 审查清单

- [ ] 仅使用 `GET` 和 `POST`。
- [ ] 列表路由是 `/list`。
- [ ] 新增和更新统一为 `/save`，并由 `Id` 区分。
- [ ] 启用、禁用路由分别为 `/enable`、`/disable`。
- [ ] 单删和批删复用 `POST /delete`，请求体为 ID 数组。
- [ ] Controller、`IXXService`、`XXService`、`Models/XXs` 均已存在且职责清晰。
- [ ] 模型未定义在 Controller 或 Service 接口文件内。
- [ ] 特殊业务规则已在服务端实现并覆盖通用规则。
- [ ] 构建、静态检查和必要接口验证已完成。
