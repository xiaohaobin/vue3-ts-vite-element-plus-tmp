# 关于环境配置和API服务器地址管理相关

本文档描述了在项目中如何管理多环境配置，以及在生产环境下处理多个 API 服务器地址（如分区域：中国区、澳洲区、全球区）的配置方案。

## 1. 基础环境配置
项目使用 Vite 的 `.env` 文件机制管理环境变量：
- `.env.development`: 开发环境
- `.env.test`: 测试环境
- `.env.production`: 生产环境

每个文件通过 `VITE_APP_API_URL` 定义基础接口路径。

## 2. 生产环境多区域 API 配置方案
当生产环境需要根据区域（中国区、澳洲区、全球区）指向不同的 API 地址时，推荐以下两种方案：

### 方案 A：通过多模式构建（推荐）
为每个区域创建独立的模式（Mode），在构建时指定对应的配置文件。

1. **新建环境文件**：
   - `.env.production.cn` (中国区)
   - `.env.production.au` (澳洲区)
   - `.env.production.global` (全球区)

2. **文件内容示例** (`.env.production.cn`):
   ```bash
   NODE_ENV = production
   VITE_APP_API_URL = https://api-cn.xxx.com
   ```

3. **修改 `package.json` 脚本**:
   ```json
   "scripts": {
     "build:cn": "vite build --mode production.cn",
     "build:au": "vite build --mode production.au",
     "build:global": "vite build --mode production.global"
   }
   ```

### 方案 B：运行时动态选择
如果需要在一个包中根据用户选择或 IP 动态切换 API，可以在代码中配置映射。

1. **在 `.env.production` 中定义所有地址**:
   ```bash
   VITE_APP_API_URL_CN = https://api-cn.xxx.com
   VITE_APP_API_URL_AU = https://api-au.xxx.com
   VITE_APP_API_URL_GLOBAL = https://api-global.xxx.com
   ```

2. **更新类型定义** ([vite-env.d.ts](file:///d:/vue3/%E6%A8%A1%E6%9D%BF/admin-element-vue-vite.ts2/src/@types/vite-env.d.ts)):
   ```typescript
   interface ImportMetaEnv {
     readonly VITE_APP_API_URL_CN: string;
     readonly VITE_APP_API_URL_AU: string;
     readonly VITE_APP_API_URL_GLOBAL: string;
   }
   ```

3. **在请求封装中使用** ([request.ts](file:///d:/vue3/%E6%A8%A1%E6%9D%BF/admin-element-vue-vite.ts2/src/utils/request.ts)):
   根据本地存储或逻辑选择基础路径：
   ```typescript
   const getBaseURL = () => {
     const region = localStorage.getItem('region') || 'global';
     const mapping = {
       cn: import.meta.env.VITE_APP_API_URL_CN,
       au: import.meta.env.VITE_APP_API_URL_AU,
       global: import.meta.env.VITE_APP_API_URL_GLOBAL
     };
     return mapping[region];
   }
   ```

## 3. 注意事项
- **绝对地址**: 生产环境下 `VITE_APP_API_URL` 通常设置为带 `https://` 的绝对地址，以跳过 Vite 的开发代理。
- **类型安全**: 新增任何 `VITE_` 开头的变量，务必同步更新 `src/@types/vite-env.d.ts` 以获得代码提示。
