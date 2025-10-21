# ByteOne 前端项目开发指导说明

## 项目概述

ByteOne 前端是一个基于 **Vue 3 + TypeScript + Naive UI** 的现代化管理后台应用，提供设备管理、打印任务、固件升级等核心功能。

## 技术栈

### 核心框架
- **Vue 3.5+** - 组合式 API (Composition API)
- **TypeScript 5.8+** - 类型安全
- **Vite 7+** - 构建工具
- **Pinia** - 状态管理
- **Vue Router 4.5+** - 路由管理

### UI 框架
- **Naive UI 2.43+** - 主要 UI 组件库
- **@vicons/ionicons5** - 图标库

### 功能库
- **vue-i18n 9.14+** - 国际化
- **axios 1.12+** - HTTP 请求
- **dayjs** - 日期处理
- **lodash-es** - 工具函数
- **echarts + vue-echarts** - 图表可视化

### 自动化
- **unplugin-auto-import** - 自动导入 API
- **unplugin-vue-components** - 自动导入组件

## 项目结构

```
pkgs/front/
├── src/
│   ├── api/                    # API 接口定义
│   │   ├── http.ts            # Axios 封装和拦截器
│   │   └── modules/           # 按模块划分的 API
│   ├── assets/                # 静态资源
│   ├── components/            # 全局组件
│   │   ├── Permission/        # 权限相关组件
│   │   └── DeviceSelector/    # 设备选择器等
│   ├── composables/           # 组合式函数
│   │   ├── usePermissions.ts  # 权限管理
│   │   └── useFileUpload.ts   # 文件上传
│   ├── config/                # 配置文件
│   │   └── theme.ts           # 主题配置
│   ├── constants/             # 常量定义
│   ├── layouts/               # 布局组件
│   │   └── AppLayout.vue      # 主布局
│   ├── locales/               # 国际化
│   │   ├── index.ts           # i18n 配置
│   │   ├── zh-CN.ts           # 中文翻译
│   │   └── en-US.ts           # 英文翻译
│   ├── plugins/               # 插件
│   │   ├── directives.ts      # 自定义指令
│   │   └── echarts.ts         # ECharts 配置
│   ├── router/                # 路由配置
│   │   └── index.ts           # 路由定义和守卫
│   ├── stores/                # Pinia 状态管理
│   │   ├── auth.ts            # 认证状态
│   │   ├── theme.ts           # 主题状态
│   │   └── ...                # 其他模块状态
│   ├── styles/                # 全局样式
│   │   └── index.css          # 主样式文件
│   ├── types/                 # TypeScript 类型定义
│   ├── utils/                 # 工具函数
│   │   ├── permission.ts      # 权限工具
│   │   ├── token.ts           # Token 管理
│   │   └── feedback.ts        # 反馈工具
│   ├── views/                 # 页面组件
│   │   ├── auth/              # 权限管理页面
│   │   ├── device/            # 设备管理页面
│   │   ├── print/             # 打印任务页面
│   │   ├── firmware/          # 固件管理页面
│   │   └── ...                # 其他模块页面
│   ├── App.vue                # 根组件
│   └── main.ts                # 入口文件
├── .gitignore
├── index.html
├── package.json
├── tsconfig.json
└── vite.config.ts
```

## 创建新模块指南

### 1. 国际化 (i18n) 配置

在创建新模块前，**必须先添加国际化翻译**。

#### 步骤 1: 添加中文翻译

编辑 `src/locales/zh-CN.ts`，在对应的模块下添加翻译：

```typescript
const zhCN = {
  // ... 其他翻译
  
  // 新模块翻译示例
  myModule: {
    entity: '我的模块',
    
    // 筛选器
    filters: {
      name: '名称',
      status: '状态',
      keyword: '关键字',
    },
    
    // 表格
    table: {
      title: '列表',
      name: '名称',
      description: '描述',
      status: '状态',
      createdAt: '创建时间',
      updatedAt: '更新时间',
    },
    
    // 表单
    form: {
      createTitle: '新建',
      editTitle: '编辑',
      name: '名称',
      description: '描述',
      status: '状态',
    },
    
    // 操作反馈
    feedback: {
      create: '创建成功',
      update: '更新成功',
      delete: '删除成功',
      deleteMulti: '批量删除成功',
      loadFailed: '加载失败',
    },
    
    // 验证规则
    validation: {
      nameRequired: '请输入名称',
      statusRequired: '请选择状态',
    },
  },
}
```

#### 步骤 2: 添加英文翻译

编辑 `src/locales/en-US.ts`，**保持相同的结构**：

```typescript
const enUS = {
  // ... other translations
  
  myModule: {
    entity: 'My Module',
    
    filters: {
      name: 'Name',
      status: 'Status',
      keyword: 'Keyword',
    },
    
    table: {
      title: 'List',
      name: 'Name',
      description: 'Description',
      status: 'Status',
      createdAt: 'Created At',
      updatedAt: 'Updated At',
    },
    
    form: {
      createTitle: 'Create',
      editTitle: 'Edit',
      name: 'Name',
      description: 'Description',
      status: 'Status',
    },
    
    feedback: {
      create: 'Created successfully',
      update: 'Updated successfully',
      delete: 'Deleted successfully',
      deleteMulti: 'Batch deleted successfully',
      loadFailed: 'Failed to load',
    },
    
    validation: {
      nameRequired: 'Please enter name',
      statusRequired: 'Please select status',
    },
  },
}
```

#### 翻译注意事项

1. **保持结构一致**: 中英文翻译的 key 结构必须完全一致
2. **使用嵌套对象**: 按功能分组（filters, table, form, feedback 等）
3. **动态参数**: 使用 `{变量名}` 格式，如 `'共 {total} 条'` / `'Total {total} items'`
4. **复用 common**: 通用操作词在 `common` 模块中定义，直接使用 `t('common.actions.new')`

### 2. API 接口定义

在 `src/api/modules/` 下创建新的 API 文件。

#### 示例: `src/api/modules/myModule.ts`

```typescript
import http from '../http'

// 定义接口类型
export interface MyModuleItem {
  id: string
  name: string
  description?: string
  status: 'active' | 'disabled'
  createdAt: string
  updatedAt: string
}

export interface MyModuleListParams {
  page?: number
  size?: number
  name?: string
  status?: string
}

export interface MyModuleListResponse {
  items: MyModuleItem[]
  total: number
}

export interface CreateMyModuleDto {
  name: string
  description?: string
  status: 'active' | 'disabled'
}

export interface UpdateMyModuleDto extends Partial<CreateMyModuleDto> {
  id: string
}

// API 方法
export const myModuleApi = {
  // 获取列表
  list(params: MyModuleListParams) {
    return http.get<MyModuleListResponse>('/my-module', { params })
  },

  // 获取详情
  getById(id: string) {
    return http.get<MyModuleItem>(`/my-module/${id}`)
  },

  // 创建
  create(data: CreateMyModuleDto) {
    return http.post<MyModuleItem>('/my-module', data)
  },

  // 更新
  update(data: UpdateMyModuleDto) {
    return http.put<MyModuleItem>(`/my-module/${data.id}`, data)
  },

  // 删除
  delete(id: string) {
    return http.delete(`/my-module/${id}`)
  },

  // 批量删除
  batchDelete(ids: string[]) {
    return http.post('/my-module/batch-delete', { ids })
  },
}
```

### 3. 页面组件开发

#### 页面结构模板

创建 `src/views/myModule/MyModuleList.vue`:

```vue
<template>
  <div class="my-module-page">
    <!-- 1. 筛选卡片 -->
    <n-card 
      class="filter-card" 
      size="small" 
      :title="t('myModule.entity')" 
      :bordered="false"
    >
      <n-form label-placement="left" inline>
        <n-space wrap>
          <!-- 筛选项 -->
          <n-form-item :label="t('myModule.filters.name')">
            <n-input
              v-model:value="filters.name"
              size="small"
              clearable
              :placeholder="t('common.placeholder.search', { target: t('myModule.filters.name') })"
            />
          </n-form-item>
          
          <n-form-item :label="t('myModule.filters.status')">
            <n-select
              v-model:value="filters.status"
              size="small"
              clearable
              :options="statusOptions"
              :placeholder="t('common.placeholder.select', { target: t('myModule.filters.status') })"
              style="width: 100px"
            />
          </n-form-item>

          <!-- 操作按钮 -->
          <n-form-item>
            <n-button 
              type="primary" 
              size="small" 
              @click="handleSearch" 
              :loading="loading"
            >
              {{ t('common.actions.apply') }}
            </n-button>
          </n-form-item>
          
          <n-form-item>
            <n-button 
              size="small" 
              @click="handleReset" 
              :disabled="loading"
            >
              {{ t('common.actions.reset') }}
            </n-button>
          </n-form-item>
        </n-space>
      </n-form>
    </n-card>

    <!-- 2. 数据表格卡片 -->
    <n-card 
      class="table-card" 
      size="small" 
      :title="t('myModule.table.title')" 
      :bordered="false"
    >
      <!-- 表格工具栏 -->
      <template #header-extra>
        <n-space>
          <!-- 使用权限按钮组件 -->
          <PermissionButton
            resource="my-module"
            action="create"
            type="primary"
            size="small"
            @click="openCreateModal"
          >
            {{ t('common.actions.new') }}
          </PermissionButton>
          
          <PermissionButton
            resource="my-module"
            action="delete"
            size="small"
            type="error"
            :disabled="!selectedRowKeys.length"
            @click="handleBatchDelete"
          >
            {{ t('common.actions.batchDelete') }}
          </PermissionButton>
        </n-space>
      </template>

      <!-- 数据表格 -->
      <n-data-table
        :columns="columns"
        :data="tableData"
        :loading="loading"
        :row-key="rowKey"
        :pagination="false"
        :bordered="false"
        checkable
        v-model:checked-row-keys="selectedRowKeys"
      >
        <template #empty>
          <n-empty :description="t('common.feedback.emptyTable')" />
        </template>
      </n-data-table>

      <!-- 分页器 -->
      <div class="table-footer">
        <n-pagination
          v-model:page="pagination.page"
          v-model:page-size="pagination.size"
          :item-count="pagination.total"
          :page-sizes="[10, 20, 50]"
          show-size-picker
          @update:page="handlePageChange"
          @update:page-size="handlePageSizeChange"
        >
          <template #suffix="{ itemCount }">
            {{ t('common.pagination.total', { total: itemCount }) }}
          </template>
        </n-pagination>
      </div>
    </n-card>

    <!-- 3. 新建/编辑模态框 -->
    <n-modal 
      v-model:show="showModal" 
      :title="modalTitle" 
      preset="card" 
      style="width: 520px"
    >
      <n-form 
        ref="formRef" 
        :model="formModel" 
        :rules="formRules" 
        label-width="96"
      >
        <n-form-item :label="t('myModule.form.name')" path="name">
          <n-input
            v-model:value="formModel.name"
            :placeholder="t('common.placeholder.input', { target: t('myModule.form.name') })"
          />
        </n-form-item>

        <n-form-item :label="t('myModule.form.description')" path="description">
          <n-input
            v-model:value="formModel.description"
            type="textarea"
            :placeholder="t('common.placeholder.input', { target: t('myModule.form.description') })"
          />
        </n-form-item>

        <n-form-item :label="t('myModule.form.status')" path="status">
          <n-radio-group v-model:value="formModel.status">
            <n-radio value="active">{{ t('common.status.active') }}</n-radio>
            <n-radio value="disabled">{{ t('common.status.disabled') }}</n-radio>
          </n-radio-group>
        </n-form-item>
      </n-form>

      <template #action>
        <n-space justify="end">
          <n-button @click="showModal = false" :disabled="saving">
            {{ t('common.actions.cancel') }}
          </n-button>
          <n-button type="primary" @click="handleSubmit" :loading="saving">
            {{ t('common.actions.confirm') }}
          </n-button>
        </n-space>
      </template>
    </n-modal>
  </div>
</template>

<script setup lang="ts">
import { ref, reactive, computed, onMounted, h } from 'vue'
import { useI18n } from 'vue-i18n'
import { useMessage, useDialog, type DataTableColumns } from 'naive-ui'
import type { FormInst, FormRules } from 'naive-ui'
import { myModuleApi, type MyModuleItem, type MyModuleListParams } from '@/api/modules/myModule'
import PermissionButton from '@/components/Permission/PermissionButton.vue'
import { usePermissions } from '@/composables/usePermissions'

// ============ 组合式 API ============
const { t } = useI18n()
const message = useMessage()
const dialog = useDialog()
const { checkResourcePermission } = usePermissions()

// ============ 响应式数据 ============
const loading = ref(false)
const saving = ref(false)
const showModal = ref(false)
const isEdit = ref(false)
const formRef = ref<FormInst | null>(null)

// 筛选器
const filters = reactive<MyModuleListParams>({
  page: 1,
  size: 10,
  name: '',
  status: undefined,
})

// 表格数据
const tableData = ref<MyModuleItem[]>([])
const selectedRowKeys = ref<string[]>([])

// 分页
const pagination = reactive({
  page: 1,
  size: 10,
  total: 0,
})

// 表单数据
const formModel = reactive({
  id: '',
  name: '',
  description: '',
  status: 'active' as 'active' | 'disabled',
})

// ============ 计算属性 ============
const modalTitle = computed(() => 
  isEdit.value ? t('myModule.form.editTitle') : t('myModule.form.createTitle')
)

const statusOptions = computed(() => [
  { label: t('common.status.active'), value: 'active' },
  { label: t('common.status.disabled'), value: 'disabled' },
])

// 表格列定义
const columns = computed<DataTableColumns<MyModuleItem>>(() => [
  {
    title: t('myModule.table.name'),
    key: 'name',
    ellipsis: { tooltip: true },
  },
  {
    title: t('myModule.table.description'),
    key: 'description',
    ellipsis: { tooltip: true },
  },
  {
    title: t('myModule.table.status'),
    key: 'status',
    render: (row) => {
      const statusMap = {
        active: t('common.status.active'),
        disabled: t('common.status.disabled'),
      }
      return statusMap[row.status] || row.status
    },
  },
  {
    title: t('common.table.actions'),
    key: 'actions',
    width: 180,
    render: (row) => {
      return h(
        'div',
        { style: 'display: flex; gap: 8px;' },
        [
          checkResourcePermission('my-module', 'update') &&
            h(
              'a',
              { onClick: () => openEditModal(row) },
              t('common.actions.edit')
            ),
          checkResourcePermission('my-module', 'delete') &&
            h(
              'a',
              { 
                onClick: () => handleDelete(row.id),
                style: 'color: var(--n-color-error);'
              },
              t('common.actions.delete')
            ),
        ].filter(Boolean)
      )
    },
  },
])

// 表单验证规则
const formRules = computed<FormRules>(() => ({
  name: [
    {
      required: true,
      message: t('myModule.validation.nameRequired'),
      trigger: 'blur',
    },
  ],
  status: [
    {
      required: true,
      message: t('myModule.validation.statusRequired'),
      trigger: 'change',
    },
  ],
}))

// ============ 方法 ============
const rowKey = (row: MyModuleItem) => row.id

// 加载数据
const loadData = async () => {
  loading.value = true
  try {
    const params = {
      ...filters,
      page: pagination.page,
      size: pagination.size,
    }
    const response = await myModuleApi.list(params)
    tableData.value = response.data?.items || []
    pagination.total = response.data?.total || 0
  } catch (error) {
    console.error('[MyModule] Load data failed:', error)
    message.error(t('myModule.feedback.loadFailed'))
  } finally {
    loading.value = false
  }
}

// 搜索
const handleSearch = () => {
  pagination.page = 1
  loadData()
}

// 重置筛选
const handleReset = () => {
  Object.assign(filters, {
    page: 1,
    size: 10,
    name: '',
    status: undefined,
  })
  handleSearch()
}

// 分页变化
const handlePageChange = (page: number) => {
  pagination.page = page
  loadData()
}

const handlePageSizeChange = (size: number) => {
  pagination.size = size
  pagination.page = 1
  loadData()
}

// 打开新建模态框
const openCreateModal = () => {
  isEdit.value = false
  Object.assign(formModel, {
    id: '',
    name: '',
    description: '',
    status: 'active',
  })
  showModal.value = true
}

// 打开编辑模态框
const openEditModal = (row: MyModuleItem) => {
  isEdit.value = true
  Object.assign(formModel, {
    id: row.id,
    name: row.name,
    description: row.description || '',
    status: row.status,
  })
  showModal.value = true
}

// 提交表单
const handleSubmit = async () => {
  if (!formRef.value) return

  try {
    await formRef.value.validate()
  } catch (error) {
    return
  }

  saving.value = true
  try {
    if (isEdit.value) {
      await myModuleApi.update(formModel)
      message.success(t('myModule.feedback.update'))
    } else {
      await myModuleApi.create(formModel)
      message.success(t('myModule.feedback.create'))
    }
    showModal.value = false
    loadData()
  } catch (error) {
    console.error('[MyModule] Submit failed:', error)
  } finally {
    saving.value = false
  }
}

// 删除单项
const handleDelete = (id: string) => {
  dialog.warning({
    title: t('common.confirm.destructive'),
    content: t('common.confirm.deleteSingle', { name: t('myModule.entity') }),
    positiveText: t('common.actions.confirm'),
    negativeText: t('common.actions.cancel'),
    onPositiveClick: async () => {
      try {
        await myModuleApi.delete(id)
        message.success(t('myModule.feedback.delete'))
        loadData()
      } catch (error) {
        console.error('[MyModule] Delete failed:', error)
      }
    },
  })
}

// 批量删除
const handleBatchDelete = () => {
  if (!selectedRowKeys.value.length) {
    message.warning(t('common.feedback.selectAtLeastOne'))
    return
  }

  dialog.warning({
    title: t('common.confirm.destructive'),
    content: t('common.confirm.deleteSelected', {
      count: selectedRowKeys.value.length,
      name: t('myModule.entity'),
    }),
    positiveText: t('common.actions.confirm'),
    negativeText: t('common.actions.cancel'),
    onPositiveClick: async () => {
      try {
        await myModuleApi.batchDelete(selectedRowKeys.value)
        message.success(t('myModule.feedback.deleteMulti'))
        selectedRowKeys.value = []
        loadData()
      } catch (error) {
        console.error('[MyModule] Batch delete failed:', error)
      }
    },
  })
}

// ============ 生命周期 ============
onMounted(() => {
  loadData()
})
</script>

<style scoped>
.my-module-page {
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 16px;
}

.filter-card,
.table-card {
  background: var(--color-bg-card);
}

.table-footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
}
</style>
```

### 4. 页面样式规范

#### 全局 CSS 变量

项目使用 CSS 变量实现主题切换，定义在 `src/styles/index.css`:

```css
:root {
  /* 主色调 */
  --color-primary: #ff7a1a;
  --color-primary-hover: #ff9242;
  --color-primary-pressed: #d96400;
  
  /* 信息色 */
  --color-info: #2f88ff;
  --color-info-hover: #4f9dff;
  --color-info-pressed: #1f6ce5;
  
  /* 背景色 */
  --color-bg-layout: #f5f7fb;
  --color-bg-card: #ffffff;
}
```

#### 页面布局样式

```vue
<style scoped>
/* 页面容器 */
.module-page {
  display: flex;
  flex-direction: column;
  gap: 16px;
  padding: 16px;
}

/* 卡片样式 */
.filter-card,
.table-card {
  background: var(--color-bg-card);
  border-radius: 12px;
}

/* 表格底部 */
.table-footer {
  display: flex;
  justify-content: flex-end;
  margin-top: 16px;
}

/* 表单项间距 */
.n-form-item {
  margin-bottom: 16px;
}
</style>
```

### 5. 主题控制

#### 主题配置

主题配置在 `src/config/theme.ts`:

```typescript
export type ThemeKey = 'vibrant-orange' | 'tech-blue'

export interface ThemePreset {
  id: ThemeKey
  labelKey: string
  descriptionKey: string
  accent: string
  accentSoft: string
  cssVars: Record<string, string>
  overrides: GlobalThemeOverrides
}
```

#### 使用主题

```vue
<script setup lang="ts">
import { useThemeStore } from '@/stores/theme'

const themeStore = useThemeStore()

// 切换主题
const switchTheme = (themeKey: ThemeKey) => {
  themeStore.applyTheme(themeKey)
}

// 获取当前主题
const currentTheme = computed(() => themeStore.current)
const themeOverrides = computed(() => themeStore.themeOverrides)
</script>

<template>
  <!-- Naive UI 根组件配置主题 -->
  <n-config-provider :theme-overrides="themeOverrides">
    <n-message-provider>
      <n-dialog-provider>
        <!-- 应用内容 -->
      </n-dialog-provider>
    </n-message-provider>
  </n-config-provider>
</template>
```

### 6. 权限控制

#### 权限组合式函数

项目使用 `usePermissions` composable 进行权限管理:

```typescript
import { usePermissions } from '@/composables/usePermissions'

const { 
  checkPermission,           // 检查完整权限字符串
  checkResourcePermission,   // 检查资源:操作权限
  getResourcePermissions,    // 获取资源的CRUD权限
  isSuperAdmin,             // 是否超级管理员
  isDealerAdmin             // 是否经销商管理员
} = usePermissions()

// 检查权限
if (checkResourcePermission('users', 'create')) {
  // 用户有创建权限
}

// 获取资源权限
const permissions = getResourcePermissions('users')
// {
//   canCreate: true,
//   canRead: true,
//   canUpdate: true,
//   canDelete: false,
// }
```

#### 权限按钮组件

使用 `PermissionButton` 组件自动根据权限显示/隐藏按钮:

```vue
<template>
  <!-- 方式 1: 使用 resource + action -->
  <PermissionButton
    resource="users"
    action="create"
    type="primary"
    @click="handleCreate"
  >
    新建用户
  </PermissionButton>

  <!-- 方式 2: 使用完整权限字符串 -->
  <PermissionButton
    permission="users:delete"
    type="error"
    @click="handleDelete"
  >
    删除用户
  </PermissionButton>
</template>
```

#### 在表格列中使用权限

```typescript
const columns = computed<DataTableColumns<User>>(() => [
  // ... 其他列
  {
    title: t('common.table.actions'),
    key: 'actions',
    render: (row) => {
      return h(
        'div',
        { style: 'display: flex; gap: 8px;' },
        [
          // 编辑按钮 - 有权限才显示
          checkResourcePermission('users', 'update') &&
            h(
              'a',
              { onClick: () => handleEdit(row) },
              t('common.actions.edit')
            ),
          
          // 删除按钮 - 有权限才显示
          checkResourcePermission('users', 'delete') &&
            h(
              'a',
              { 
                onClick: () => handleDelete(row.id),
                style: 'color: var(--n-color-error);'
              },
              t('common.actions.delete')
            ),
        ].filter(Boolean) // 过滤掉 false 值
      )
    },
  },
])
```

### 7. 路由配置

#### 添加新路由

编辑 `src/router/index.ts`:

```typescript
const routes: RouteRecordRaw[] = [
  {
    path: '/',
    component: () => import('@/layouts/AppLayout.vue'),
    children: [
      // 添加新模块路由
      {
        path: 'my-module/list',
        name: 'MyModuleList',
        component: () => import('@/views/myModule/MyModuleList.vue'),
        meta: { title: 'routes.myModuleList' }, // i18n key
      },
    ],
  },
]
```

#### 路由元信息

```typescript
interface RouteMeta {
  title: string          // 页面标题的 i18n key
  public?: boolean       // 是否公开页面（不需要登录）
}
```

### 8. 状态管理 (Pinia)

#### 创建 Store

在 `src/stores/` 下创建新的 store:

```typescript
// src/stores/myModule.ts
import { defineStore } from 'pinia'
import { myModuleApi, type MyModuleItem } from '@/api/modules/myModule'

export const useMyModuleStore = defineStore('myModule', {
  state: () => ({
    items: [] as MyModuleItem[],
    loading: false,
    currentItem: null as MyModuleItem | null,
  }),

  getters: {
    activeItems: (state) => state.items.filter(item => item.status === 'active'),
    itemCount: (state) => state.items.length,
  },

  actions: {
    async fetchItems() {
      this.loading = true
      try {
        const response = await myModuleApi.list({ page: 1, size: 100 })
        this.items = response.data?.items || []
      } catch (error) {
        console.error('[MyModuleStore] Fetch items failed:', error)
        throw error
      } finally {
        this.loading = false
      }
    },

    async fetchItemById(id: string) {
      try {
        const response = await myModuleApi.getById(id)
        this.currentItem = response.data || null
        return this.currentItem
      } catch (error) {
        console.error('[MyModuleStore] Fetch item failed:', error)
        throw error
      }
    },

    clearCurrentItem() {
      this.currentItem = null
    },
  },
})
```

#### 在组件中使用 Store

```vue
<script setup lang="ts">
import { useMyModuleStore } from '@/stores/myModule'

const myModuleStore = useMyModuleStore()

// 访问状态
const items = computed(() => myModuleStore.items)
const loading = computed(() => myModuleStore.loading)

// 调用 action
onMounted(async () => {
  await myModuleStore.fetchItems()
})
</script>
```

## 开发规范

### 1. 组件命名

- **页面组件**: PascalCase，如 `UserList.vue`, `DeviceDetail.vue`
- **通用组件**: PascalCase，如 `PermissionButton.vue`, `DeviceSelector.vue`
- **布局组件**: PascalCase with 前缀，如 `AppLayout.vue`, `AppHeader.vue`

### 2. 文件命名

- **TypeScript 文件**: camelCase，如 `usePermissions.ts`, `myModule.ts`
- **Vue 文件**: PascalCase，如 `UserList.vue`
- **配置文件**: kebab-case，如 `vite.config.ts`, `tsconfig.json`

### 3. 代码风格

#### 导入顺序

```typescript
// 1. Vue 相关
import { ref, reactive, computed, onMounted } from 'vue'
import { useRouter, useRoute } from 'vue-router'
import { useI18n } from 'vue-i18n'

// 2. UI 库
import { useMessage, useDialog } from 'naive-ui'
import type { DataTableColumns } from 'naive-ui'

// 3. API 和类型
import { myModuleApi, type MyModuleItem } from '@/api/modules/myModule'

// 4. 组件
import PermissionButton from '@/components/Permission/PermissionButton.vue'

// 5. Composables
import { usePermissions } from '@/composables/usePermissions'

// 6. 工具函数
import { formatDate } from '@/utils/date'
```

#### 响应式数据组织

```typescript
// ============ 组合式 API ============
const { t } = useI18n()
const message = useMessage()
const router = useRouter()

// ============ 响应式数据 ============
const loading = ref(false)
const showModal = ref(false)

const filters = reactive({
  keyword: '',
  status: undefined,
})

// ============ 计算属性 ============
const filteredData = computed(() => {
  // ...
})

// ============ 方法 ============
const handleSearch = () => {
  // ...
}

// ============ 生命周期 ============
onMounted(() => {
  // ...
})
```

### 4. TypeScript 类型定义

#### 优先使用类型推导

```typescript
// ✅ Good - 类型推导
const count = ref(0)
const items = ref<MyModuleItem[]>([])

// ❌ Bad - 不必要的类型标注
const count: Ref<number> = ref(0)
```

#### 为复杂对象定义类型

```typescript
// ✅ Good - 定义接口
interface FormModel {
  name: string
  description?: string
  status: 'active' | 'disabled'
}

const formModel = reactive<FormModel>({
  name: '',
  description: '',
  status: 'active',
})
```

### 5. 错误处理

#### API 调用错误处理

```typescript
const loadData = async () => {
  loading.value = true
  try {
    const response = await myModuleApi.list(params)
    tableData.value = response.data?.items || []
  } catch (error) {
    // 记录日志 - 使用模块前缀
    console.error('[MyModule] Load data failed:', error)
    // 用户提示 - 已在 http.ts 拦截器中处理
    // message.error(t('myModule.feedback.loadFailed'))
  } finally {
    loading.value = false
  }
}
```

#### 表单验证错误处理

```typescript
const handleSubmit = async () => {
  if (!formRef.value) return

  try {
    // 验证表单
    await formRef.value.validate()
  } catch (error) {
    // 验证失败，自动显示错误信息
    return
  }

  // 继续提交逻辑
  saving.value = true
  try {
    await myModuleApi.create(formModel)
    message.success(t('myModule.feedback.create'))
    showModal.value = false
  } catch (error) {
    console.error('[MyModule] Submit failed:', error)
  } finally {
    saving.value = false
  }
}
```

### 6. 性能优化

#### 使用 computed 缓存计算结果

```typescript
// ✅ Good
const filteredItems = computed(() => 
  items.value.filter(item => item.status === 'active')
)

// ❌ Bad - 每次渲染都重新计算
const getFilteredItems = () => 
  items.value.filter(item => item.status === 'active')
```

#### 列表渲染使用 key

```vue
<!-- ✅ Good -->
<div v-for="item in items" :key="item.id">
  {{ item.name }}
</div>

<!-- ❌ Bad -->
<div v-for="(item, index) in items" :key="index">
  {{ item.name }}
</div>
```

#### 组件懒加载

```typescript
// 路由懒加载
{
  path: 'my-module/list',
  component: () => import('@/views/myModule/MyModuleList.vue'),
}

// 组件懒加载
const HeavyComponent = defineAsyncComponent(() => 
  import('@/components/HeavyComponent.vue')
)
```

### 7. 可访问性 (A11y)

#### 表单标签

```vue
<!-- ✅ Good - 使用 label -->
<n-form-item :label="t('myModule.form.name')" path="name">
  <n-input v-model:value="formModel.name" />
</n-form-item>

<!-- ❌ Bad - 没有 label -->
<n-input v-model:value="formModel.name" />
```

#### 按钮文本

```vue
<!-- ✅ Good - 有明确文本 -->
<n-button @click="handleDelete">
  {{ t('common.actions.delete') }}
</n-button>

<!-- ⚠️ OK - 图标按钮配合 tooltip -->
<n-tooltip :content="t('common.actions.delete')">
  <n-button circle @click="handleDelete">
    <n-icon :component="TrashOutline" />
  </n-button>
</n-tooltip>
```

### 8. 调试技巧

#### 添加调试日志

```typescript
// 使用模块前缀，便于过滤
console.log('[MyModule] Loading data...')
console.error('[MyModule] Failed to load:', error)
console.warn('[MyModule] Deprecated feature used')
```

#### Vue Devtools

- 安装 Vue Devtools 浏览器扩展
- 检查组件树、状态、事件
- 监控 Pinia store 变化

#### 网络请求调试

- 浏览器 Network 面板查看 API 请求
- 检查 Request Headers (Authorization token)
- 检查 Response (code, message, data)

## 常见问题

### 1. i18n 翻译不显示

**问题**: 页面显示翻译 key 而不是实际文本

**解决方案**:
1. 检查 `zh-CN.ts` 和 `en-US.ts` 是否都添加了翻译
2. 确认 key 路径正确，如 `myModule.table.name`
3. 重启开发服务器

### 2. 权限按钮不显示

**问题**: `PermissionButton` 组件没有显示

**解决方案**:
1. 检查用户菜单是否包含该权限
2. 确认权限字符串格式正确（`resource:action`）
3. 超级管理员应该能看到所有按钮

### 3. API 请求 401 错误

**问题**: API 请求返回 401 Unauthorized

**解决方案**:
1. 检查 token 是否过期
2. 查看浏览器 Network 面板的 Authorization header
3. 重新登录获取新 token

### 4. Naive UI 组件样式异常

**问题**: 组件样式显示不正确

**解决方案**:
1. 确保在 `App.vue` 中配置了 `n-config-provider`
2. 检查是否正确传入 `theme-overrides`
3. 查看浏览器控制台是否有 CSS 警告

### 5. 自动导入不工作

**问题**: Vue API 或组件需要手动导入

**解决方案**:
1. 检查 `vite.config.ts` 中的 `AutoImport` 和 `Components` 插件配置
2. 删除 `node_modules` 和 `.vite` 缓存，重新安装依赖
3. 重启 IDE/编辑器以更新类型提示

## 最佳实践总结

### ✅ 推荐做法

1. **优先使用组合式 API** - 使用 `<script setup>` 语法
2. **完整的 i18n 支持** - 所有用户可见文本都使用 `t()` 函数
3. **TypeScript 类型安全** - 为 API 响应和表单数据定义类型
4. **权限控制** - 使用 `PermissionButton` 和权限检查函数
5. **错误处理** - 完善的 try-catch 和用户反馈
6. **代码注释** - 关键逻辑添加清晰注释
7. **响应式设计** - 考虑不同屏幕尺寸
8. **性能优化** - 使用 computed、懒加载、虚拟列表

### ❌ 避免的做法

1. **硬编码文本** - 不要直接写中文/英文，使用 i18n
2. **忽略类型定义** - 不要使用 `any` 类型
3. **全局状态滥用** - 非共享状态不要放到 Pinia
4. **直接操作 DOM** - 使用 Vue 的响应式系统
5. **忽略错误处理** - 所有 async 操作都要 try-catch
6. **过度嵌套** - 组件拆分要合理
7. **重复代码** - 提取公共逻辑到 composables
8. **未清理的副作用** - 记得在 `onUnmounted` 中清理

## 快速参考

### 常用 Composables

```typescript
// 国际化
import { useI18n } from 'vue-i18n'
const { t, locale } = useI18n()

// 权限
import { usePermissions } from '@/composables/usePermissions'
const { checkResourcePermission, isSuperAdmin } = usePermissions()

// 路由
import { useRouter, useRoute } from 'vue-router'
const router = useRouter()
const route = useRoute()

// Naive UI 消息
import { useMessage, useDialog } from 'naive-ui'
const message = useMessage()
const dialog = useDialog()
```

### 常用工具函数

```typescript
// 日期格式化
import { formatDate } from '@/utils/date'
formatDate(date, 'YYYY-MM-DD HH:mm:ss')

// Token 管理
import { getToken, setToken, removeToken } from '@/utils/token'

// 枚举转换
import { getEnumLabel } from '@/utils/enum'
```

### Naive UI 常用组件

```vue
<!-- 卡片 -->
<n-card :title="title" :bordered="false" size="small">
  <template #header-extra>工具栏</template>
  内容
</n-card>

<!-- 表单 -->
<n-form :model="form" :rules="rules" ref="formRef">
  <n-form-item label="名称" path="name">
    <n-input v-model:value="form.name" />
  </n-form-item>
</n-form>

<!-- 数据表格 -->
<n-data-table
  :columns="columns"
  :data="data"
  :loading="loading"
  :pagination="false"
  checkable
  v-model:checked-row-keys="selected"
/>

<!-- 模态框 -->
<n-modal v-model:show="visible" preset="card" :title="title">
  内容
  <template #action>
    <n-button @click="visible = false">取消</n-button>
    <n-button type="primary" @click="handleConfirm">确认</n-button>
  </template>
</n-modal>
```

---

## 总结

本文档涵盖了 ByteOne 前端项目的核心开发规范和最佳实践。在创建新模块时，请：

1. **先添加 i18n 翻译**（中英文）
2. **定义 API 接口和类型**
3. **创建页面组件**（参考模板）
4. **配置路由**
5. **实现权限控制**
6. **编写样式**（使用 CSS 变量）
7. **测试功能**（多语言、权限、主题切换）

遵循这些规范，可以确保代码质量、可维护性和团队协作效率。
