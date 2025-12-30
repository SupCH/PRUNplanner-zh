# PRUNplanner 中文版

[![Status](https://img.shields.io/badge/%E7%8A%B6%E6%80%81-%E9%9D%9E%E5%AE%8C%E6%95%B4%E7%89%88-orange)](https://github.com/SupCH/PRUNplanner-zh)
[![Deployed on Cloudflare Pages](https://img.shields.io/badge/Cloudflare-Deployed-blue?logo=cloudflare)](https://prunp.crazzy.cn/)

欢迎使用 PRUNplanner 中文版！本项目是基于 [PRUNplanner 官方前端](https://github.com/PRUNplanner/frontend) 的汉化版本，旨在为 Prosperous Universe 的中国玩家提供更友好的界面体验。

> [!CAUTION]
> **注意：** 本项目目前处于**非完整汉化版**阶段。大部分核心页面（主页、管理、帝国、搜索、登录注册）已完成汉化，但部分二级工具和细节内容仍可能显示为英文。我们正在持续改进中。

## 🚀 在线预览

您可以直接访问部署在 Cloudflare Pages 上的版本：
**[https://prunp.crazzy.cn/](https://prunp.crazzy.cn/)**

## ✨ 功能特性

- **深度汉化**: 覆盖主页、帝国管理、计划详情、星球搜索、加载界面及登录注册等核心流程。
- **中英切换**: 导航栏内置语言切换功能，支持随时在中文与原版英文间无缝切换。
- **UI 组件本地化**: 完美适配 Naive UI 组件库的中文环境。
- **保留原味**: 严格保留官方 API 地址、项目名称、游戏名称、阵营缩写等专有名词为英文。

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
| VITE_SHARE_BASE_URL | string | "https://prunplanner.org/shared" | 计划分享链接基础地址 |

## ⚖️ 许可证

本项目遵循 [MIT License](LICENSE)。

---
*Made with &hearts; by SupCH and PRUNplanner contributors.*
