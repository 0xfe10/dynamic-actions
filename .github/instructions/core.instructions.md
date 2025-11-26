---
applyTo: "pkgs/core/**"
description: "ByteOne 后端项目开发指导说明"
---

# ByteOne 后端项目开发提示词

## 项目概览
ByteOne 是基于 GoFrame v2.9 的企业级物联网打印平台，支持多租户隔离。

## 🚨 核心编码规则

### 代码生成规范
- **修改结构体别忘记同步API层的结构体定义** ：每次修改数据结构后，确保同步更新对应API层的请求和响应结构体定义
- **代码复用**：先扫描现有项目代码，优先复用已有模块代码，避免重复生成相似业务逻辑
- **生成模块时考虑架构**：按照分层架构，每次生成业务模块时候要考虑代码架构和职责划分
- **依赖依赖原则**：依赖其他业务模块时候，先检查业务是否已经实现，如果已经实现就直接引用，不要生成新的重复业务。如果业务不存在则先创建相应模块使用默认值和注释占位，待依赖模块实现后再完善
- **模块完整性**：确保生成的单个模块在当前阶段可以独立编译和运行
- **渐进式开发**：优先实现核心业务逻辑，依赖模块后续补充完善
- **service层类型定义**：如果logic层需要使用到新的类型，需要在service层定义接口和类型或者在model层定义类型，如果在service层定义类型，则去除掉service层顶部的自动化生成注释
- **避免硬编码**：所有状态字段必须使用枚举常量，禁止直接使用字符串
- **错误处理**：禁止直接返回字符串错误，必须使用统一错误码。对于底层错误必须使用 `gerror.WrapCode` 包裹，不要直接使用 `gerror.New`，上层错误直接返回
- **错误打印**：使用 `g.Log().Errorf("%+v", err)` 打印错误时候，必须使用 `%+v` 以便打印完整堆栈
- **时间字段自动管理**：`created_at`、`updated_at` 由GoFrame框架自动管理，Logic层不需要手动设置
- **禁止生成测试代码**：不允许生成单元测试代码
- **生成代码后编译测试**：每次生成代码后必须编译测试，确保代码可以正常编译运行
- **数据库迁移文件**： 每次修改数据库表结构后必须编写SQL迁移文件，放在 `pkgs/core/manifest/migrations/` 目录，down和up分别对应升级和回滚SQL，文件名开头必须是递增数字
- **数据库索引**： 只添加业务查询必须的索引，避免过多冗余索引影响写入性能
- **不需要运行服务**：生成代码后不需要运行服务，默认已经调试运行了。确保代码可以编译通过即可
- **禁止生成总结文档**： 不允许生成任何总结文档，直接生成代码即可

```go
// 示例：依赖处理方式
func (s *sDevice) GetFirmwareInfo(ctx context.Context, deviceId string) (info *FirmwareInfo, err error) {
    // TODO: 依赖固件模块，待实现后补充
    // 当前返回默认值避免编译错误
    return &FirmwareInfo{
        Version: "unknown", // 默认值
        Status:  "pending", // 默认状态
    }, nil
    
    // FIXME: 需要实现firmware模块后完善此逻辑
    // return service.Firmware().GetDeviceFirmwareInfo(ctx, deviceId)
}
```

### 代码生成流程
1. 生成DAO层
   - 运行 `gf gen dao` 生成DAO和Model（数据层）
2. 生成API层
   - 由AI生成API定义，放在 `pkgs/core/api/[模块名]/v1/` 目录
   - 包含请求和响应结构体
3. 生成ctrl层
   - 运行 `gf gen ctrl` 生成Controller接口以及api层定义文件
4. 生成logic层
   - 由AI生成Logic层代码
5. 生成service层
   - 运行 `gf gen service` 生成Service接口
6. 生成枚举映射
   - 运行 `gf gen enums` 生成枚举映射
7. 实现控制层
   - 由AI修改Controller实现，每个API方法一个文件
8. 添加路由
   - 在 `pkgs/core/internal/router/router.go` 中添加路由

### 文件组织规范
#### API层文件结构
```
pkgs/core/api/
  [模块名]/
    [模块名].go              # 自动生成的接口定义
    v1/
      [模块名].go            # v1版本API定义，包含请求/响应结构体
```

#### Controller层文件结构

以下结构会自动生成，只需修改对应控制器方法就行
```
pkgs/core/internal/controller/
  [模块名]/
    [模块名].go              # 空文件，自动生成
    [模块名]_new.go          # 控制器构造函数
    [模块名]_v1_[方法名].go   # 每个API方法一个文件
```

### 错误处理规范
- **统一错误码**：所有错误使用 `gerror.NewCode(code.GetErrorCodeByCode(code.XxxCode))` 包裹
- **错误码定义**：在 `pkgs/core/internal/code/code.go` 中按模块分组定义，如：
  - 用户相关：5001-5099
  - 角色相关：5090-5099  
  - 设备相关：5600-5699
  - 打印任务：5800-5899
- **错误映射**：每个错误码必须在 `ErrorMsgMap` 中提供中文消息

#### 标准错误处理模式
```go
// 数据库操作错误
gerror.NewCode(code.GetErrorCodeByCode(code.DatabaseInsertFailCode), "创建失败")
gerror.NewCode(code.GetErrorCodeByCode(code.DatabaseUpdateFailCode), "更新失败")
gerror.NewCode(code.GetErrorCodeByCode(code.DatabaseQueryFailCode), "查询失败")

// 业务逻辑错误
gerror.NewCode(code.GetErrorCodeByCode(code.DataNotFoundCode), "数据不存在")
gerror.NewCode(code.GetErrorCodeByCode(code.DataAlreadyExistsCode), "数据已存在")

// 包装错误
gerror.WrapCode(code.GetErrorCodeByCode(code.DatabaseQueryFailCode), err, "查询失败")
```

### 路由注册规范
```go
// 在 pkgs/core/internal/router/router.go 中添加路由组
[模块名]V1 := v1.Group("/api/v1").Middleware(
    service.Middleware().CORS,
    service.Middleware().Ctx,
    service.Middleware().ResponseHandler,
    service.Middleware().Auth,
)
[模块名]V1.Bind([模块名].NewV1())
```

### 枚举字段规范
- **类型定义**：使用字符串常量类型，定义在 `pkgs/core/internal/consts/common.go`
- **命名规范**：`TypeName + 具体值`，如 `StateActive`、`PrintJobStateQueued`
- **注释要求**：每个枚举类型和常量都必须有中文注释

```go
// 示例：状态枚举
type State string // 代表对象的状态

const (
	StateActive   State = "active"   // 激活状态
	StateDisabled State = "disabled" // 禁用状态
)
```

### API层复用规范
- **必须复用common字段**：使用 `pkgs/core/api/common/v1/common.go` 中定义的通用结构
- **分页字段**：`*commonV1.PageReq`（请求）、`*commonV1.PageRes`（响应）
- **排序字段**：`*commonV1.SortReq`（排序字段和方向）
- **时间范围**：`*commonV1.CreateTimeRangeReq`（开始时间、结束时间）
- **审计信息**：`*commonV1.CreateOrUpdateInfoRes`（创建人、修改人、时间等）
- **枚举字段**：必须使用 `pkgs/core/internal/consts` 中定义的枚举类型，如 `consts.State`，并且使用goframe的验证标签`v:"enums"`进行校验

#### API基础结构体模式
```go
// 基础结构体，包含公共字段
type [模块名]Base struct {
    Field1 string `json:"field1" description:"字段描述" v:"required#字段是必须的"`
    Field2 string `json:"field2" description:"字段描述" v:"length:1,50#字段长度限制"`
}

// 创建请求
type [模块名]CreateReq struct {
    g.Meta `path:"/[模块路径]" method:"post" summary:"创建[模块]" tags:"[模块]管理"`
    [模块名]Base
}

type [模块名]CreateRes struct {
    Id string `json:"id" description:"[模块]ID"`
}

// 更新请求
type [模块名]UpdateReq struct {
    g.Meta `path:"/[模块路径]/{id}" method:"put" summary:"更新[模块]" tags:"[模块]管理"`
    Id     string `json:"id" in:"path" v:"required#[模块]ID不能为空"`
    [模块名]Base
}

type [模块名]UpdateRes struct{}
```

#### API请求验证规范
- 必填字段使用`v:"required#错误消息"`
- 长度限制使用`v:"length:1,50#错误消息"`
- 枚举验证使用`v:"enums#错误消息"`
- 路径参数使用`in:"path"`

```go
// API结构体示例
type UserListReq struct {
    g.Meta   `path:"/users" method:"get" summary:"用户列表" tags:"用户管理"`
    Username string       `json:"username" description:"用户名"`
    State    consts.State `json:"state" description:"状态" v:"enums"`
    *commonV1.PageReq    // 复用分页字段
    *commonV1.SortReq    // 复用排序字段
}

type UserGetRes struct {
    Id       string `json:"id" description:"用户ID"`
    Username string `json:"username" description:"用户名"`
    *commonV1.CreateOrUpdateInfoRes // 复用审计信息
}
```

## 🏗️ 分层架构职责

### Controller层（业务组装）
- ✅ **每个API方法一个文件**：文件命名格式 `[模块名]_v1_[方法名].go`
- ✅ **简洁直接**：控制器只做参数转换和服务调用，不包含复杂业务逻辑
- ✅ 获取用户上下文：`userCtx := utility.GetUserContext(ctx)`
- ✅ 数据验证：使用对应的工具函数检查当前用户是否有权限操作
- ✅ 业务判断：检查重复、权限验证
- ✅ 数据转换：`gconv.Struct(req, &entity)`
- ✅ 设置审计字段：`CreatedBy`、`DealerId` 等
- ✅ **统一调用模式**：`service.[模块名]().[方法名](ctx, params)`
- ✅ **标准响应构造**：严格按照API定义返回响应结构

#### Controller方法标准模式
```go
// 文件: [模块名]_v1_[方法名].go
package [模块名]

import (
    "context"
    v1 "byteone/pkgs/core/api/[模块名]/v1"
    "byteone/pkgs/core/internal/service"
    "byteone/pkgs/core/internal/model/do"
    "github.com/gogf/gf/v2/util/gconv"
)

```go
package [模块名]

import (
    v1 "byteone/pkgs/core/api/[模块名]/v1"
    "byteone/pkgs/core/internal/service"
    "byteone/pkgs/core/internal/model/do"
    "github.com/gogf/gf/v2/util/gconv"
)

func (c *ControllerV1) [模块名][方法名](ctx context.Context, req *v1.[请求类型]) (res *v1.[响应类型], err error) {
    // 1. 数据转换(如需要)
    var data *do.[模块名]
    err = gconv.Struct(req, &data)
    if err != nil {
        return nil, err
    }

    // 2. 调用service层
    result, err := service.[模块名]().[方法名](ctx, data)
    if err != nil {
        return nil, err
    }
    err = gconv.Struct(result, &res)
    return
}
```

#### Controller构造函数
```go
// 文件: [模块名]_new.go
package [模块名]

import "byteone/pkgs/core/api/[模块名]"

type ControllerV1 struct{}

func NewV1() [模块名].I[模块名]V1 {
    return &ControllerV1{}
}
```

### Logic层（基础业务）
- ✅ **标准服务注册**：使用 `init()` 函数和 `New()` 构造函数注册服务
- ✅ **权限控制方法**：涉及数据查询的方法使用WithPermission后缀
- ✅ **用户信息获取**：通过 `utility.GetUserContext(ctx)` 获取当前用户
- ✅ **ID生成**：统一使用 `guid.S()` 生成唯一标识
- ✅ 基础CRUD操作
- ✅ 数据校验和业务规则
- ✅ 可选的多租户隔离参数：`dealerId ...string`
- ❌ **不设置时间字段**：`created_at`、`updated_at` 由GoFrame框架自动管理，Logic层不需要手动设置
- ✅ **单模块实现**：每次只实现当前模块的核心逻辑，对其他模块的依赖使用注释和默认值处理
- ✅ **必须使用Common层通用函数**：
  - **分页排序**：`service.Common().GetTotalWithPageWithSort(db, req.PageReq, req.SortReq)` - 用户分页和排序以及总数查询
  - **权限过滤**：`service.Common().FilterPermissionData(ctx, db)` - 自动处理多租户和用户权限
  - **权限判断**：`service.Common().HasEditPermission(ctx, req)` - 检查修改权限
  - **时间范围查询**：`service.Common().AddCreateTimeRangeReqToDb(db, req.CreateTimeRangeReq)`

#### Service层接口定义标准格式
```go
type I[模块名] interface {
    // Create 创建[模块]
    Create(ctx context.Context, data *do.[模块名]) (id string, err error)
    // UpdateWithPermission 在当前用户权限范围内更新[模块]
    UpdateWithPermission(ctx context.Context, data *do.[模块名]) error
    // GetById 根据ID获取[模块]
    GetById(ctx context.Context, id string) (*entity.[模块名], error)
    // Delete 删除[模块]
    Delete(ctx context.Context, ids []string) error
    // ListWithPermission 在当前用户权限范围内获取[模块]列表
    ListWithPermission(ctx context.Context, req *v1.[模块名]ListReq) (res *v1.[模块名]ListRes, err error)
}

var (
    local[模块名] I[模块名]
)

func [模块名]() I[模块名] {
    if local[模块名] == nil {
        panic("implement not found for interface I[模块名], forgot register?")
    }
    return local[模块名]
}

func Register[模块名](i I[模块名]) {
    local[模块名] = i
}
```

#### Logic层标准结构
```go
package [模块名]

import (
    v1 "byteone/pkgs/core/api/[模块名]/v1"
    "byteone/pkgs/core/internal/code"
    "byteone/pkgs/core/internal/consts"
    "byteone/pkgs/core/internal/dao"
    "byteone/pkgs/core/internal/model/do"
    "byteone/pkgs/core/internal/model/entity"
    "byteone/pkgs/core/internal/service"
    "byteone/pkgs/core/utility"
    "context"
    "github.com/gogf/gf/v2/errors/gerror"
    "github.com/gogf/gf/v2/frame/g"
    "github.com/gogf/gf/v2/os/gtime"
    "github.com/gogf/gf/v2/util/gconv"
    "github.com/gogf/gf/v2/util/guid"
)

type s[模块名] struct{}

func init() {
    service.Register[模块名](New())
}

func New() service.I[模块名] {
    return &s[模块名]{}
}
```

#### Create方法标准模式
```go
func (s *s[模块名]) Create(ctx context.Context, data *do.[模块名]) (id string, err error) {
    // 1. 获取用户信息
    user := utility.GetUserContext(ctx)
    
    // 2. 业务验证（如唯一性检查等）
    exists, _, err := s.CheckSomeConstraint(ctx, someField)
    if err != nil {
        return "", err
    }
    if exists {
        return "", gerror.NewCode(code.GetErrorCodeByCode(code.DataAlreadyExistsCode), "数据已存在")
    }
    
    // 3. 生成ID和设置基础字段
    id = guid.S()
    data.Id = id
    data.DealerId = user.DealerId
    data.CreatedBy = user.UserId
    data.CreatedByName = user.Username
    
    // 4. 业务逻辑处理
    // ...
    
    // 5. 插入数据库
    result, err := dao.[模块名].Ctx(ctx).Data(data).Insert()
    if err != nil {
        return "", gerror.NewCode(code.GetErrorCodeByCode(code.DatabaseInsertFailCode), "创建失败")
    }
    if rows, _ := result.RowsAffected(); rows < 1 {
        return "", gerror.NewCode(code.GetErrorCodeByCode(code.DatabaseInsertFailCode), "创建失败,没有任何修改")
    }
    return id, nil
}
```

#### UpdateWithPermission方法标准模式
```go
func (s *s[模块名]) UpdateWithPermission(ctx context.Context, data *do.[模块名]) error {
    // 1. 获取当前用户信息
    user := utility.GetUserContext(ctx)

    // 对传参类外键id进行验证
    b, err := service.DeviceModel().CheckExists(ctx, data.ModelId)
    if err != nil {
        return err
    }
    if !b {
        return gerror.NewCode(code.GetErrorCodeByCode(code.DeviceModelNotFoundCode), "设备型号不存在")
    }

    // 2. 获取原有数据进行验证
    existing, err := s.GetById(ctx, gconv.String(data.Id))
    if err != nil {
        return err
    }
    if existing == nil {
        return gerror.NewCode(code.GetErrorCodeByCode(code.DataNotFoundCode), "数据不存在")
    }

    // 3. 业务验证逻辑
    // ...

    // 4. 设置更新信息
    data.UpdatedBy = user.UserId
    data.UpdatedByName = user.Username

    // 5. 构建查询条件和权限过滤
    query := dao.[模块名].Ctx(ctx).Where(dao.[模块名].Columns().Id, data.Id)
    if !user.IsDealerAdmin {
        query = query.Where(dao.[模块名].Columns().DealerId, user.DealerId)
    }

    // 6. 执行更新
    result, err := query.Data(data).Update()
    if err != nil {
        return gerror.NewCode(code.GetErrorCodeByCode(code.DatabaseUpdateFailCode), "更新失败")
    }
    if rows, _ := result.RowsAffected(); rows < 1 {
        return gerror.NewCode(code.GetErrorCodeByCode(code.DataNotFoundCode), "数据不存在或无权限")
    }
    return nil
}
```

```go
// Logic层标准查询模式
func (s *sUser) List(ctx context.Context, req *v1.UserListReq, dealerId ...string) (res *v1.UserListRes, err error) {
    err = g.DB().Transaction(ctx, func(ctx context.Context, tx gdb.TX) error {
        res = &v1.UserListRes{}
        db := dao.User.Ctx(ctx)
        
        // 2. 应用权限过滤 (自动处理超管、租户管理员、普通用户权限)
        db = service.Common().FilterPermissionData(ctx, db)
        
        // 3. 应用业务筛选条件
        if req.Username != "" {
            db = db.WhereLike(dao.User.Columns().Username, "%"+req.Username+"%")
        }
        
        // 4. 应用时间范围查询
        db = service.Common().AddCreateTimeRangeReqToDb(db, req.CreateTimeRangeReq)
        
        // 5. 应用分页和排序 (必须使用通用函数)
        db, res.PageRes, err = service.Common().GetTotalWithPageWithSort(db, req.PageReq, req.SortReq)
        if err != nil {
            return err
        }
        
        // 6. 执行查询
        return db.Scan(&res.List)
    })
    return
}
```

## GoFrame 静态模型关联规范

### Model层静态关联模型创建
用于需要关联查询其他表数据时，通过ORM `with` 标签实现静态关联，避免N+1查询问题。

#### 标准关联模型结构
```go
// 文件: pkgs/core/internal/model/[模块名].go
package model

import (
    "github.com/gogf/gf/v2/os/gtime"
)

// [模块名]WithRelation 带关联的[模块名]模型
type [模块名]WithRelation struct {
    g.Meta `orm:"table:b_[表名]"` // 必须：指定主表名
    
    // 主表字段（显式展开所有字段，不使用entity嵌入）
    Id              string      `orm:"id" json:"id"`
    DealerId        string      `orm:"dealer_id" json:"dealerId"`
    RelationId      string      `orm:"relation_id" json:"relationId"` // 关联外键
    FieldName       string      `orm:"field_name" json:"fieldName"`
    CreatedAt       *gtime.Time `orm:"created_at" json:"createdAt"`
    UpdatedAt       *gtime.Time `orm:"updated_at" json:"updatedAt"`
    
    // 关联字段（使用model层类型，不使用entity）
    RelationData *Model[关联模块名]Out `orm:"with:relation_id=id" json:"relationData"` // 必须：使用with标签指定关联关系
}
```

**关键要点：**
- ✅ 必须包含 `g.Meta` 标签指定表名
- ✅ 显式展开主表所有字段（不使用entity嵌入）
- ✅ 关联字段使用model层类型（如 `ModelDeviceModelOut`），不使用entity类型
- ✅ 使用 `orm:"with:外键字段=关联表主键"` 标签定义关联关系
- ❌ 不使用 `entity.[模块名]` 嵌入方式
- ❌ 关联字段不使用 `entity.[关联模块]` 类型

### Logic层使用静态关联
使用 `g.Model()` 和 `WithAll()` 实现关联数据加载。

#### 标准查询模式
```go
func (s *s[模块名]) ListWithPermission(ctx context.Context, req *v1.[模块名]ListReq) (res *v1.[模块名]ListRes, err error) {
    res = &v1.[模块名]ListRes{}
    
    // 1. 使用静态关联模型构建查询
    query := g.Model(model.[模块名]WithRelation{}).Ctx(ctx).WithAll()
    cls := dao.[模块名].Columns()
    
    // 2. 添加查询条件
    if req.SomeField != "" {
        query = query.Where(cls.SomeField, req.SomeField)
    }
    
    // 3. 分页和排序
    query, res.PageRes, err = service.Common().GetTotalWithPageWithSort(query, req.PageReq, req.SortReq)
    if err != nil {
        return nil, gerror.NewCode(code.GetErrorCodeByCode(code.DatabaseQueryFailCode), "查询失败")
    }
    
    // 4. 扫描到关联模型数组
    var list []*model.[模块名]WithRelation
    err = query.Scan(&list)
    if err != nil {
        return nil, gerror.NewCode(code.GetErrorCodeByCode(code.DatabaseQueryFailCode), "查询失败")
    }
    
    // 5. 转换为API响应结构
    err = gconv.Scan(list, &res.List)
    return
}
```

**关键要点：**
- ✅ 使用 `g.Model(model.[模块名]WithRelation{})` 而非 `dao.[模块名]`
- ✅ 调用 `.WithAll()` 启用所有静态关联加载
- ✅ 定义 `var list []*model.[模块名]WithRelation` 接收查询结果
- ✅ 使用 `gconv.Scan(list, &res.List)` 转换为API响应

### API层使用关联数据
API响应结构中使用嵌套结构体表示关联数据。

#### 标准响应结构
```go
// 文件: pkgs/core/api/[模块名]/v1/[模块名].go
package v1

import (
    [关联模块]V1 "github.com/prtmax/byteone/core/api/[关联模块]/v1"
)

type [模块名]GetRes struct {
    Id         string `json:"id" description:"主键ID"`
    FieldName  string `json:"fieldName" description:"字段描述"`
    
    // 关联数据（使用关联模块的GetRes结构）
    RelationData *[关联模块]V1.[关联模块]GetRes `json:"relationData" description:"关联数据"`
    
    [模块名]Base
    *commonV1.CreateOrUpdateInfoRes
}
```

**关键要点：**
- ✅ 导入关联模块的API包（使用别名避免冲突）
- ✅ 使用关联模块的 `[关联模块]GetRes` 作为嵌套字段类型
- ✅ 字段名与model层关联字段名保持一致（如 `RelationData`）
- ❌ 不直接使用其他模块的 `Req` 类型作为响应字段

### 完整示例参考
```go
// Model层: pkgs/core/internal/model/firmware_package.go
type FirmwarePackageWithModel struct {
    g.Meta `orm:"table:b_firmware_packages"`
    
    Id           string      `orm:"id" json:"id"`
    ModelId      string      `orm:"model_id" json:"modelId"`
    Version      string      `orm:"version" json:"version"`
    // ... 其他字段
    
    DeviceModel *ModelDeviceModelOut `orm:"with:model_id=id" json:"deviceModel"`
}

// Logic层: pkgs/core/internal/logic/firmware_packages/firmware_packages.go
func (s *sFirmwarePackages) ListWithPermission(ctx context.Context, req *v1.FirmwarePackageListReq) (*v1.FirmwarePackageListRes, error) {
    res := &v1.FirmwarePackageListRes{}
    query := g.Model(model.FirmwarePackageWithModel{}).Ctx(ctx).WithAll()
    
    var list []*model.FirmwarePackageWithModel
    err = query.Scan(&list)
    err = gconv.Scan(list, &res.List)
    return res, err
}

// API层: pkgs/core/api/firmware_packages/v1/firmware_packages.go
type FirmwarePackageGetRes struct {
    Id          string                          `json:"id"`
    DeviceModel *deviceModelV1.DeviceModelGetRes `json:"deviceModel"`
    FirmwarePackageBase
}
```

## 数据库设计规范

### 索引规范
- 必须索引：`dealer_id`、主要查询字段组合

## IoT 特定模式

### 设备通讯协议
- 协议文档：`wiki/device-protocol.md`、`wiki/ayn-iot-protocol.md`

## 开发命令与工具

```bash
# 核心生成命令 (需要在 pkgs/core/ 目录下执行)
cd pkgs/core/
gf gen dao      # 生成DAO和Model（数据层）
gf gen service  # 生成Service接口（服务层）
gf gen ctrl     # 生成Controller（控制层）
gf gen enums    # 生成枚举映射
```

## 关键文件参考
- **错误码**：`pkgs/core/internal/code/code.go`
- **枚举常量**：`pkgs/core/internal/consts/common.go`
- **API公共字段**：`pkgs/core/api/common/v1/common.go` - 分页、排序、审计等通用结构
- **Logic通用函数**：`pkgs/core/internal/logic/common/common.go` - 分页排序、权限过滤、时间范围查询
- **用户上下文**：`pkgs/core/utility/context.go`
- **数据库工具**：`pkgs/core/utility/database.go`
- **JWT管理**：`pkgs/core/utility/jwt.go`
- **数据库迁移**：`pkgs/core/manifest/migrations/` - SQL迁移文件目录
- **DAO层示例**：`pkgs/core/internal/dao/user.go`、`pkgs/core/internal/dao/device.go`
- **Entity模型**：`pkgs/core/internal/model/entity/` - 数据库实体定义
- **DO模型**：`pkgs/core/internal/model/do/` - 数据操作对象

## 特别注意事项
- **单模块开发**：每次只生成一个完整模块，确保当前模块可独立编译运行
- **依赖模块处理**：对未实现的依赖模块使用注释说明和默认值，避免编译错误
- **多租户隔离**：所有业务查询必须包含 `dealer_id` 过滤
- **不生成测试**：禁止生成单元测试代码
- **错误处理**：必须使用统一错误码，不允许直接返回字符串错误
- **枚举使用**：状态字段必须使用预定义枚举常量，不允许硬编码字符串
- **时间字段自动管理**：GoFrame框架自动处理 `created_at`、`updated_at` 字段，代码中无需手动设置
- **API字段复用**：必须使用 `pkgs/core/api/common/v1` 中的通用字段，避免重复定义分页、排序、审计等结构

## 用户编码习惯特点
1. **简洁明了**: 控制器方法直接调用service层，不做复杂逻辑
2. **职责分离**: 每个API方法一个文件，职责清晰，便于维护
3. **统一的错误处理**: 直接返回service层的错误，不在控制器层处理
4. **标准的响应格式**: 严格按照API定义构造响应结构体
5. **权限意识**: 方法名包含WithPermission，体现权限控制思想
6. **中文注释**: 代码注释和错误信息使用中文，清晰易懂
7. **模板化开发**: 使用标准的代码模式和结构，确保一致性
8. **ID生成标准**: 统一使用 `guid.S()` 生成唯一标识
9. **用户上下文获取**: 通过 `utility.GetUserContext(ctx)` 获取当前用户信息
10. **数据转换**: 使用 `gconv.Struct` 进行结构体之间的数据转换

## 开发步骤标准流程
1. 分析数据库表结构，了解字段定义和关系
2. 创建API定义(v1包)，包含所有请求/响应结构体
3. 实现Logic层业务逻辑，包含所有CRUD方法
4. 实现Controller层方法，每个API一个文件
5. 在router.go中注册路由
6. 测试编译和基本功能验证

---
严格遵循 GoFrame v2.9 开发规范和多租户架构模式。
