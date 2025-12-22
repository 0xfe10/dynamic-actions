---
applyTo: "core/**"
description: "endfather -> Go(core) 重构开发指导（GoFrame v2.9.6）"
---

# endfather -> Go(core) 重构开发提示词

## 注意事项

- 重构后的go代码尽量使用GoFrame框架内部提供的功能和方法，避免引入不必要的第三方库，以保持代码的一致性和可维护性。
- 遵循GoFrame的最佳实践和编码规范，确保代码质量和性能。

## 目标与范围
本仓库正在使用 Go 语言重构/替换 endfather（原 Java/Spring 多模块体系）。本指令文件只约束 `core/**` 下的 Go 代码改动，目标是：

- 以契约与边界为核心，逐步迁移 endfather 的模块能力到 Go。
- 保持现有行为可对齐/可回归：接口输入输出、错误语义、关键业务规则不“悄悄变更”。
- 每次改动保持工程可编译、可运行（至少 `go test ./...` 能通过编译）。

## 当前工程事实（写代码前先对齐）
- Go 模块名：`core`（见 [core/go.mod](core/go.mod)）。
- GoFrame 版本：`github.com/gogf/gf/v2 v2.9.6`。
- HTTP 入口：
  - 进程入口在 [core/main.go](core/main.go)
  - Server 启动与 OpenAPI 通用响应配置在 [core/internal/cmd/cmd.go](core/internal/cmd/cmd.go)
- 路由注册聚合点： [core/internal/router/router.go](core/internal/router/router.go)
- 通用响应结构： [core/internal/model/http/http.go](core/internal/model/http/http.go)
- 中间件注册与实现：
  - service 接口在 [core/internal/service/middleware.go](core/internal/service/middleware.go)
  - 具体实现与注册在 [core/internal/logic/middleware/middleware.go](core/internal/logic/middleware/middleware.go)
- DAO/Model 生成配置： [core/hack/config.yaml](core/hack/config.yaml)（gfcli.gen.dao）

## 重构总原则（面向 endfather 迁移）
1) 契约先行：先明确 Java 侧的 DTO/接口语义，再落到 Go 的 API/DTO/Service 接口。
2) 分层清晰：把“约束/接口”“实现”“适配器/SDK”“入口”拆开，避免把 SDK 调用塞进 controller。
3) 适配器隔离：外设/第三方（OSS/短信/识别/打印等）必须通过 adapter 包封装，便于替换与灰度。
4) 渐进式替换：允许先做最小可用版本（stub/默认值/只读接口），但必须显式标注 TODO/FIXME，且不破坏编译。
5) 不做大扫除：避免为“顺手好看”重命名/重排大量文件；重构要围绕迁移目标最小化变更。

## endfather 模块到 core 的映射建议
结合 [wiki/refactor-analysis.md](wiki/refactor-analysis.md) 的分层思想，建议在 Go 侧按以下方式落地（不是硬性，但要保持一致）：

- endfather `commons/types-*`：映射为 Go 的 DTO/领域类型（优先放在 `core/internal/model/...` 或 `core/api/...`）。
- endfather `constraints`：映射为 Go 的 `service` 接口（`core/internal/service`），体现“契约”。
- endfather `accomplishes`：映射为 Go 的 `logic` 实现（`core/internal/logic/<domain>`），并在 `init()` 中 `service.RegisterXxx(New())`。
- endfather `components`：按粒度拆到 `logic`（领域服务）+ `utility`（通用能力）+ `controller`（HTTP 组装）。
- endfather `remotes`：映射为 Go 的 adapter/client（建议 `core/internal/logic/<domain>/adapter` 或 `core/utility/<xxx>`）。
- endfather `launchers`：映射为 Go 的入口与子命令（`core/internal/cmd`）。

## 目录与文件组织（以本仓库为准）
建议保持与现有骨架一致：

```
core/
  api/                      # 对外 API 结构体（req/res），按域拆分
  internal/
    cmd/                    # 进程入口/命令
    router/                 # 路由绑定
    controller/             # HTTP 控制器（组装、参数转换、调用 service）
    service/                # 领域接口（契约）+ RegisterXxx
    logic/                  # 领域实现（实现 service 接口）
    dao/                    # gf gen dao 生成的数据访问层
    model/                  # entity/do + 领域 DTO
    packed/                 # gf 打包资源/配置（现状已有引用）
  utility/                  # 通用工具（尽量无业务语义）
```

## 编码规则（重构场景最重要的几条）
### 1) 先查再写
- 写新能力前先全局搜索是否已有相似实现（尤其是 `utility/`、`internal/logic/` 里）。
- 迁移 endfather 时，优先复刻契约与边界，而不是一口气“顺便优化”。

### 2) 错误处理与返回码
本项目当前通用响应是：`{ err, message, data }`（见 [core/internal/model/http/http.go](core/internal/model/http/http.go)），并通过 GoFrame 的 error code（`gerror.Code(err)`）映射到 `err` 字段（见 [core/internal/logic/middleware/middleware.go](core/internal/logic/middleware/middleware.go)）。

- 禁止用 `fmt.Errorf("...")`/`errors.New("...")` 直接裸抛“无 code”的错误（会导致统一响应退化为内部错误）。
- 推荐：
  - 参数/状态类错误：`return gerror.NewCode(gcode.CodeInvalidParameter, "xxx")`
  - 资源不存在：`return gerror.NewCode(gcode.CodeNotFound, "xxx")`
  - 权限问题：`return gerror.NewCode(gcode.CodeNotAuthorized, "xxx")`
  - 包装底层错误：`return gerror.WrapCode(gcode.CodeInternalError, err, "xxx")`
- 打日志时如需堆栈：用 `g.Log().Errorf(ctx, "%+v", err)`（确保 `%+v`）。

### 3) DAO/Model 使用规范
- `core/internal/dao` 与 `core/internal/model/entity|do` 多为生成代码：
  - 不要在 `dao/internal/**` 里手改（会被覆盖）。
  - 业务逻辑放在 `internal/logic/**`，数据访问用 dao 组装条件。
- 结构体字段与表字段变化：必须同步更新生成物（`gf gen dao`）并确保编译通过。

### 4) 依赖未迁移模块的处理方式
允许分阶段迁移：当依赖的子域尚未迁移到 Go，可用“接口先行 + 默认实现占位”的方式保证编译。

```go
// 示例：先定义契约（service），实现可先 stub
// TODO/FIXME 必须写清楚：何时替换、对齐哪个 endfather 模块/方法。
```

### 5) Controller/Router 约束
- Controller 只做：参数校验/转换、调用 `service.Xxx()`、返回结果。
- 路由统一在 [core/internal/router/router.go](core/internal/router/router.go) 聚合注册，按 `/api/v1` 分组。
- 通用中间件按 [core/internal/cmd/cmd.go](core/internal/cmd/cmd.go) 当前做法挂载（跨域 + `ghttp.MiddlewareHandlerResponse`）。

## GoFrame CLI（与本仓库路径对齐）
如果需要重新生成 dao/service/ctrl，命令需要在 `core/` 目录下执行：

```bash
cd core
gf gen dao
gf gen service
gf gen ctrl
```

提示：生成文件通常有 “Code generated… DO NOT EDIT.” 头注释；若确需手工维护，可按提示删除自动化注释并承担后续不再覆盖生成的成本。

## 基本自检（每次提交前至少做到）
- `go test ./...`（用于编译校验；若未来加入测试也会一并跑起来）
- 涉及路由/响应变更时：确认 OpenAPI 通用响应仍为 `{err,message,data}`，并且 `err` 来自 `gerror.Code(err)`。

---
一句话：按 endfather 的“契约/实现/适配/入口”边界迁移到 `core`，保持可编译、可替换、可灰度。
