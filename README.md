# PRUNplanner 中文版

[![Status](https://img.shields.io/badge/%E7%8A%B6%E6%80%81-%E5%B7%B2%E5%AE%8C%E6%88%90-brightgreen)](https://github.com/SupCH/PRUNplanner-zh)
[![Deployed on Cloudflare Pages](https://img.shields.io/badge/Cloudflare-Deployed-blue?logo=cloudflare)](https://prunp.crazzy.cn/)

欢迎使用 PRUNplanner 中文版！本项目是 [PRUNplanner 官方前端](https://github.com/PRUNplanner/frontend) 的深度汉化与优化版本，旨在为 Prosperous Universe 的中国玩家提供完整且流畅的本地化体验。

## 🚀 在线预览

您可以直接访问部署在 Cloudflare Pages 上的版本：
**[https://prunp.crazzy.cn/](https://prunp.crazzy.cn/)**

## ✨ 功能特性

- **深度全面汉化**: 覆盖 100% 的用户界面，包括所有二级工具（ROI 分析、购物车、生产链可视化、总部升级计算器等）。
- **动态语言切换**: 支持在中文与原版英文间无缝切换，所有翻译均通过 `vue-i18n` 标准化管理。
- **本地化日期系统**: 基于 `dayjs` 实现了相对时间的中文显示，并修复了原始项目中 FIO 数据更新时间显示异常的 Bug。
- **全量帮助文档汉化**: 汉化了全部 12+ 份功能指南文档（Markdown），并支持随界面语言动态切换内容。
- **自定义分享链接**: 优化了计划分享功能，默认生成的分享链接现已指向本项目域名 `prunp.crazzy.cn`，方便中国玩家直接查看。
- **UI 深度优化**: 为右侧 Material I/O 面板添加了滚动支持，优化了加载界面的翻译逻辑，并修复了侧边栏 FIO 状态重叠等细节 UI Bug。
- **健壮的路由系统**: 解决了深层链接刷新时的 404 错误，确保分享体验与原版一致。
- **UI 组件适配**: 深度定制了 Naive UI 组件库的占位符与提示信息，确保护持一致的中文体验。
- **保留核心概念**: 严格保留项目原名、游戏专有名词（如 FIO, CX, Ticker 等）及阵营名称，维持与国际社区的无障碍交流。

## 🛠️ 安装与运行

### 使用 Docker (推荐)

本项目提供了预配置的 Docker 部署方案，适合本地私有化部署：

1. **克隆仓库**:
   ```bash
   git clone https://github.com/SupCH/PRUNplanner-zh.git
   cd PRUNplanner-zh
   ```

2. **构建并运行**:
   ```bash
   docker build -t prunplanner-zh:latest .
   docker compose up -d
   ```
   启动后可通过 `http://localhost:8082` 访问。

### 本地开发

1. **安装依赖**:
   ```bash
   pnpm install
   ```

2. **启动开发服务器**:
   ```bash
   pnpm run dev
   ```

## 🌐 环境变量

| 变量名 | 类型 | 默认值 | 描述 |
| --- | --- | --- | --- |
| VITE_API_BASE_URL | string | "https://api.prunplanner.org" | 官方 API 接口地址 |
| VITE_SHARE_BASE_URL | string | "https://prunp.crazzy.cn/shared" | 计划分享链接基础地址 |

## ⚖️ 声明

本项目为 [PRUNplanner](https://github.com/PRUNplanner/frontend) 的中文本地化分叉版本。原项目的版权归原作者所有。汉化及功能增强部分由 **SupCH** 维护。

## 许可证

本项目遵循 [MIT License](LICENSE)。

---
*Made with &hearts; by SupCH and PRUNplanner contributors.*