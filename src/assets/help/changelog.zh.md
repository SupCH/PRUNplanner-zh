# 2025-11-13 - v. 0.20.6

- 现有计划默认关闭自动居住建筑优化 [[PR-305]](https://github.com/PRUNplanner/frontend/pull/305)
- 在计划中添加了 “选择生产建筑” 占位符 [[PR-306]](https://github.com/PRUNplanner/frontend/pull/306)
- 修复：核心模块面积 (+25) 现在已计入单位面积利润计算 [[PR-307]](https://github.com/PRUNplanner/frontend/pull/307)
- 物料图块支持加载成交量并显示 1 天和 7 天数据 [[PR-308]](https://github.com/PRUNplanner/frontend/pull/308)

# 2025-10-26 - v. 0.20.5

- 允许在保存计划前关闭自动居住优化 [[PR-298]](https://github.com/PRUNplanner/frontend/pull/298)
- 修复：密码重置端点 URL 现可正确将邮件重置代码注入前端 [[#301]](https://github.com/PRUNplanner/frontend/issues/301)
- 如果居住建筑已自动优化，则停止向分析系统发送跟踪事件 [[#297]](https://github.com/PRUNplanner/frontend/issues/297)

# 2025-10-22 - v. 0.20.4

- 为选择器下拉列表中的所有配方添加了基于最佳配置的单位面积利润计算，同时在计划概览中添加了该指标 [[PR-290]](https://github.com/PRUNplanner/frontend/pull/290)
- 将物料和星系列表更新至最新游戏版本 [[PR-292]](https://github.com/PRUNplanner/frontend/pull/292)
- 优化了计划布局、配置栏、保存按钮和建筑面积展示 [[PR-295]](https://github.com/PRUNplanner/frontend/pull/295)
- 细微的 UI 和响应式改进

# 2025-09-25 - v. 0.20.3

- 修复了基础设施或专家无法设置为 0 的错误 [[#277]](https://github.com/PRUNplanner/frontend/pull/277)
- 从 ROI 概览表中移除了产出利润 [[PR-280]](https://github.com/PRUNplanner/frontend/pull/280)
- 使用自定义组件替换了更多 Naive UI 组件：表格 [[#273]](https://github.com/PRUNplanner/frontend/pull/273)，图标 [[#275]](https://github.com/PRUNplanner/frontend/pull/275)
- 在首页添加了对 PostHog 和 Highcharts 的致谢 [[#274]](https://github.com/PRUNplanner/frontend/pull/274)

# 2025-09-22 - v. 0.20.2

- 隔离应用版本与数据库版本，以优化新版本发布时的表现 [[#263]](https://github.com/PRUNplanner/frontend/issues/263)
- 增强了建筑购物车：物料现已单独列出，并显示 FIO 库存 [[#252]](https://github.com/PRUNplanner/frontend/issues/252)
- 防止在创建时由于无效 COGC 填充计划数据 [[#261]](https://github.com/PRUNplanner/frontend/issues/261)
- 维修分析现在可以显示选定日期的计划物料需求 [[#269]](https://github.com/PRUNplanner/frontend/issues/269)
- 在启用 FIO 时为物料购物车添加了提示 [[PR-#271]](https://github.com/PRUNplanner/frontend/pull/271)
- 改进了市场探索图表中的数据清洗逻辑 [[#193]](https://github.com/PRUNplanner/frontend/issues/193)
- 资源 / 配方 ROI：新增单位面积利润指标，并将计算并行度提升至 64 [[#207]](https://github.com/PRUNplanner/frontend/issues/207)

# 2025-09-15 - v. 0.20.1

- 对按钮、复选框和 XIT 转账操作进行了 UI 微调
- 优雅地跳过未知配方
- 取消了物料图块上的数量精度限制
- 在 XIT 消耗货运匹配中添加了 500/500 初始飞船
- 减少了 PostHog API 调用的跟踪
- 按产出代码字母顺序对计划配方选项进行排序
