<!--
Sync Impact Report
- Version change: template -> 1.0.0
- Modified principles:
	- Principle 1 -> I. 属性语义优先
	- Principle 2 -> II. 克制计算可验证
	- Principle 3 -> III. 查询体验第一
	- Principle 4 -> IV. 数据可追溯与可演进
	- Principle 5 -> V. 质量闸门与发布纪律
- Added sections:
	- 技术与产品约束
	- 开发流程与交付标准
- Removed sections: none
- Templates requiring updates:
	- ✅ .specify/templates/plan-template.md (reviewed, no change needed)
	- ✅ .specify/templates/spec-template.md (reviewed, no change needed)
	- ✅ .specify/templates/tasks-template.md (reviewed, no change needed)
	- ✅ .specify/templates/constitution-template.md (source template kept as-is)
- Follow-up TODOs: none
-->

# 属性克制图鉴 Web 项目 Constitution

## Core Principles

### I. 属性语义优先
属性、克制倍率、组合规则必须由单一规范定义并集中管理。UI 展示、筛选逻辑、
统计图表、文案解释必须共享同一份领域模型，禁止在组件内硬编码“例外规则”。
当玩法规则与显示文案冲突时，以领域规则为准并在界面显式说明原因。

### II. 克制计算可验证
克制计算结果必须是确定性的：相同输入得到相同输出，且可通过测试复现。单属性
对双属性、双属性对单属性、免疫与抗性叠加必须有显式算法和边界定义。每次调整
倍率或规则前，必须补充回归用例，覆盖至少一个正常值、一个边界值、一个冲突值。

### III. 查询体验第一
核心用户路径是“选择属性 -> 立即得到克制结果 -> 理解原因”。首屏必须可用，交互
反馈不依赖重型动画。主要查询在常见设备上应保持流畅，交互延迟目标为 p95 < 100ms。
移动端与桌面端都必须保证可读性和触控可达性。

### IV. 数据可追溯与可演进
每条属性关系数据都必须标注来源版本和更新时间。数据结构变更必须包含迁移策略或
向后兼容说明，避免历史配置失效。新增世代、地区形态或规则变体时，必须通过扩展
字段或版本层实现，禁止破坏既有查询接口。

### V. 质量闸门与发布纪律
任何影响克制结果、排序逻辑、筛选逻辑的变更都必须通过自动化测试与人工验收清单。
发布前必须完成类型检查、构建验证和关键路径手测。未通过闸门的变更不得合并到主干。

## 技术与产品约束

- 前端技术栈以 Vue 3 + Vite 为主，优先使用组合式 API 组织领域逻辑。
- 图表组件仅负责可视化，不承担业务判定；业务判定必须在独立模块实现。
- 所有对外展示的倍率、克制标签、描述文案必须支持统一配置与后续本地化。
- 禁止引入与当前需求无关的复杂状态管理或服务层，遵循最小可行复杂度。
- 构建产物必须可在静态托管环境运行，不依赖服务端渲染前提。

## 开发流程与交付标准

1. 需求建模：先明确属性表、倍率矩阵、查询输入输出，再进入界面开发。
2. 测试先行：先写克制计算与格式化输出测试，再实现或重构核心算法。
3. 分层实现：先领域模块，再状态与组合函数，最后组件与样式。
4. 合并前检查：必须通过 `npm run build`，并验证关键交互路径。
5. 交付说明：PR 必须附带变更范围、测试结果、已知限制与后续计划。

## Governance

本 Constitution 高于临时约定与个人习惯。任何原则变更都必须在 PR 中说明动机、影响
范围和迁移方案，并至少经 1 名维护者批准后生效。版本采用语义化规则：新增原则或
重大扩展升 MINOR；原则重定义或破坏兼容升 MAJOR；仅澄清文案升 PATCH。每次规划、
评审、发布都必须执行一次合规检查，确认当前工作未违反核心原则。

**Version**: 1.0.0 | **Ratified**: 2026-04-08 | **Last Amended**: 2026-04-08
