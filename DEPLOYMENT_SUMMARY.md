# PRUNplanner 前端部署与汉化总结

## 1. 部署信息
- **项目路径**: `D:\tools\PRUNplanner-frontend`
- **访问地址**: [http://localhost:8082](http://localhost:8082)
- **容器名称**: `prunplanner-frontend`
- **基础镜像**: 本地构建的 `prunplanner-frontend:latest`

## 2. 汉化实现 (i18n)
- **依赖库**: 引入了 `vue-i18n@9` 国际化框架。
- **语言包**: 
  - 路径: `src/locales/`
  - 文件: `zh.json` 和 `en.json`（基于 `D:\tools\PRUNplanner_locales_backup` 恢复）。
- **已汉化内容**:
  - **导航栏**: 全部的侧边栏菜单项（规划、工具、账户等）。
  - **UI 组件**: 通过 `AppProvider.vue` 同步了 Naive UI 组件库的中文语言包。
- **切换功能**: 在导航栏“账户”菜单底部添加了“中/英文切换”按钮。

## 3. 关键修改点
- **`main.ts`**: 注册并初始化 i18n 插件。
- **`NavigationBar.vue`**: 将硬编码字符串替换为 `$t()` 函数，并集成了语言切换逻辑。
- **`AppProvider.vue`**: 配置 `NConfigProvider` 使其支持响应式的语言切换。
- **`docker-compose.yaml`**: 
  - 端口映射修改为 `8082:80`。
  - 修改为使用本地构建镜像。
- **`.dockerignore`**: 增加了排除项（`node_modules`, `dist`, `.git`），构建提速约 80%。

## 4. 日常维护命令
- **重新构建并生效修改**:
  ```powershell
  docker build -t prunplanner-frontend:latest .
  docker compose up -d
  ```
- **更新词条**: 
  如需添加或修改汉化内容，请编辑 `src/locales/zh.json`，然后重新构建镜像。

---
*最后更新时间: 2025年12月31日*
