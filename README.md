# PRUNplanner 中文版

欢迎使用 PRUNplanner 中文版！本项目是基于 [PRUNplanner 官方前端](https://github.com/PRUNplanner/frontend) 的汉化版本，旨在为 Prosperous Universe 的中国玩家提供更友好的界面体验。

本项目通过集成 `vue-i18n` 实现了全站汉化，同时严格保留了官方的专有名词（如项目名、游戏名、阵营名等）为英文。

- **GitHub 链接**: [https://github.com/SupCH/PRUNplanner-zh](https://github.com/SupCH/PRUNplanner-zh)
- **开发者**: [SupCH](https://github.com/SupCH)
- **原始项目**: [PRUNplanner Frontend](https://github.com/PRUNplanner/frontend)

## 功能特性

- **全站汉化**: 包括主页、帝国管理、计划详情、星球搜索等核心页面。
- **中英切换**: 导航栏内置语言切换功能，可随时在中文与原版英文间切换。
- **UI 组件本地化**: 同步汉化了 Naive UI 组件库。
- **保留原味**: 所有的官方地址、名称、阵营等专有名词均维持英文原版。

## 安装与运行

### 使用 Docker (推荐)

本项目提供了预配置的 Docker 部署方案：

1. **克隆仓库**:
   ```bash
   git clone https://github.com/SupCH/PRUNplanner-zh.git
   cd PRUNplanner-zh
   ```

2. **构建镜像**:
   ```bash
   docker build -t prunplanner-zh:latest .
   ```

3. **启动容器**:
   ```bash
   docker compose up -d
   ```
   默认映射端口为 `8082`，启动后可通过 `http://localhost:8082` 访问。

### 本地开发

1. **安装依赖**:
   ```bash
   pnpm install
   ```

2. **启动开发服务器**:
   ```bash
   pnpm run dev
   ```

3. **构建生产版本**:
   ```bash
   pnpm run build
   ```

## 环境变量

| 变量名 | 类型 | 默认值 | 描述 |
| --- | --- | --- | --- |
| VITE_API_BASE_URL | string | "https://api.prunplanner.org" | 官方 API 地址 |
| VITE_SHARE_BASE_URL | string | "https://prunplanner.org/shared" | 分享基础地址 |

## 许可证

本项目遵循 [MIT License](LICENSE)。

---
*Made with &hearts; by SupCH and PRUNplanner contributors.*