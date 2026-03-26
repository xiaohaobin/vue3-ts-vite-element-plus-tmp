# 项目架构与技术使用说明

## 项目概览
该项目为基于 Vite + Vue 3 + Pinia + Element Plus 的后台管理模板，内置动态路由、权限控制、国际化与 Mock 支持。主要入口位于 `src/main.ts`，整体采用多布局（`SecurityLayout` + `MemberLayout` + `UserLayout`）组织页面结构，并通过路由元信息驱动菜单、权限与面包屑。

## 技术栈
- 构建与语言：Vite、TypeScript
- 框架与路由：Vue 3、Vue Router
- 状态管理：Pinia
- UI 组件：Element Plus
- 网络请求：Axios（封装于 `src/utils/request.ts`）
- 进度条：NProgress
- 图标方案：`vite-plugin-svg-icons`（SVG Sprite）
- Mock：`vite-plugin-mock`
- 质量与规范：ESLint、Prettier、Stylelint、Husky、lint-staged

## 目录结构（核心）
```
├─ mock/                         # Mock 数据
├─ public/                       # 静态资源（不参与构建处理）
├─ src/
│  ├─ @types/                    # 全局类型声明
│  ├─ assets/                    # 样式与图标
│  ├─ components/                # 通用组件
│  ├─ composables/               # 组合式逻辑（主题/布局/标题/国际化）
│  ├─ config/                    # 全局配置与入口（router/store/settings）
│  ├─ directives/                # 自定义指令（权限）
│  ├─ enums/                     # 枚举定义
│  ├─ layouts/                   # 布局容器（Security/Member/User）
│  ├─ locales/                   # 国际化语言包
│  ├─ pages/                     # 业务页面
│  ├─ services/                  # 接口服务
│  ├─ store/                     # Pinia store
│  ├─ utils/                     # 工具与请求封装
│  ├─ App.vue                    # 根组件
│  └─ main.ts                    # 入口
├─ vite.config.ts                # Vite 配置
└─ package.json                  # 项目信息与脚本
```

## 架构与运行流程
### 启动与应用装配
- `src/main.ts` 创建应用并注册 `router`、`pinia`、`ElementPlus`，挂载到 `#app`。
- `src/App.vue` 统一设置主题、菜单布局、菜单风格，并基于 `ElConfigProvider` 注入 Element Plus 的多语言包。

### 路由与布局
- 路由入口 `src/config/router.ts` 使用 `createWebHashHistory`。
- 布局层级：
  - `SecurityLayout` 作为最外层安全布局。
  - `MemberLayout` 作为主后台布局（含菜单/多标签等）。
  - `UserLayout` 作为登录布局。
- 业务路由在 `src/layouts/MemberLayout/routes.ts` 与 `src/layouts/UserLayout/routes.ts` 中维护。
- 路由元信息（`meta`）驱动菜单标题、图标、缓存、权限、面包屑等能力。

### 权限控制
- 页面级权限主要由路由 `meta.roles` 控制（菜单/路由展示）。
- 指令级权限由 `v-permission` 实现，结合 `src/utils/router.ts` 的 `hasPermissionRoles` 校验当前用户角色。

### 请求与鉴权
- `src/utils/request.ts` 封装 Axios：
  - 统一注入 Token（从 `localStorage` 获取）。
  - 统一错误处理与提示。
  - 请求取消机制避免重复请求。
- Token 的本地存取由 `src/utils/localToken.ts` 管理。
- 用户信息由 `src/store/user.ts` 驱动，并通过 `src/services/user.ts` 获取。

### 状态管理
- `src/config/store.ts` 初始化 Pinia。
- `src/store/global.ts` 管理布局/主题等全局状态。
- `src/store/user.ts` 管理用户信息与登录态。
- `src/store/i18n.ts` 管理语言包与 Element Plus 语言切换。

### 国际化
- 语言包位于 `src/locales/`。
- `src/utils/i18n.ts` 负责获取/设置本地语言与 HTML `lang`。
- `src/composables/useI18n.ts` 提供组件内便捷使用方式。

### 主题与布局样式
- 全局样式入口：`src/assets/css/global.scss`。
- Element Plus 样式覆盖：`src/assets/css/element-plus.scss`。
- 主题/菜单布局切换由 `src/composables/useTheme.ts`、`useMenuLayout.ts`、`useMenuStyle.ts` 驱动。

## 关键配置说明
### 站点配置
`src/config/settings.ts` 提供站点级配置（首页、主题、菜单布局、Token Key、请求白名单等）。

### Vite 与构建
`vite.config.ts` 集成：
- SVG 图标插件（`vite-plugin-svg-icons`）
- ESLint 插件（`vite-plugin-eslint`）
- Mock 插件（`vite-plugin-mock`）
- Gzip 压缩（`vite-plugin-compression`）
- 依赖可视化（`rollup-plugin-visualizer`）

> 关键环境变量（通过 `import.meta.env` 读取）：
> - `VITE_APP_PORT`：开发端口
> - `VITE_APP_MOCK`：Mock 开关
> - `VITE_APP_REPORT`：包分析开关
> - `VITE_APP_BUILD_GZIP`：构建 Gzip 开关
> - `VITE_APP_OPEN`：启动时是否打开浏览器
> - `VITE_APP_API_URL_PROXY`：开发代理目标
> - `VITE_APP_API_URL`：接口前缀

## 常用脚本
- 开发：`pnpm dev`
- 构建：`pnpm build` / `pnpm build:dev` / `pnpm build:test`
- 预览：`pnpm preview`
- SVG 生成：`pnpm svgo`
- 代码规范：`pnpm lint:eslint` / `pnpm lint:prettier` / `pnpm lint:stylelint`

## 开发约定建议
- 业务页面放在 `src/pages/`，对应路由在 `layouts/*/routes.ts` 内注册。
- 通用组件放在 `src/components/`，复用逻辑放在 `src/composables/`。
- 需要权限控制的按钮/操作使用 `v-permission` 指令。
- 新增接口统一走 `src/services/` 与 `src/utils/request.ts` 的封装。
