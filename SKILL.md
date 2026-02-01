---
name: n8n-workflow-assistant
description: "n8n 工作流自动化设计助手。帮助用户设计、创建、验证和优化 n8n 工作流。使用 n8n-MCP 与 n8n 实例交互，使用 n8n-skills 知识库查找节点和参考配置。适用场景：创建新工作流、分析需求并选择合适节点、验证工作流配置、优化现有工作流性能。关键词：n8n, workflow, automation, 工作流设计, 自动化。"
license: MIT
metadata:
  author: 阿豪
  version: "0.1.0"
---

# n8n Workflow Assistant

帮助用户设计、创建和优化 n8n 工作流的专业助手。

## 技术栈

本 Skill 依赖两个核心组件：

| 组件 | 定位 | 说明 |
|------|------|------|
| **n8n-MCP** | 实时操作引擎 | 通过 MCP 工具与 n8n 实例交互 |
| **n8n-skills** | 离线知识库 | 545 个节点文档 + 30 个社区包 + 20+ 模板 |

## 核心能力

### 可立即使用（无需 API Key）

| 功能 | MCP 工具 | 说明 |
|------|----------|------|
| 搜索节点 | `search_nodes` | 按关键词搜索 1000+ 节点 |
| 节点文档 | `get_node` | 获取节点配置、属性、示例 |
| 搜索模板 | `search_templates` | 搜索 2700+ 工作流模板 |
| 获取模板 | `get_template` | 获取完整模板 JSON |
| 验证节点 | `validate_node` | 验证节点配置 |
| 验证工作流 | `validate_workflow` | 验证完整工作流 |

### 需要 API Key（连接 n8n 实例）

| 功能 | MCP 工具 | 说明 |
|------|----------|------|
| 创建工作流 | `n8n_create_workflow` | 在 n8n 中创建新工作流 |
| 测试工作流 | `n8n_test_workflow` | 触发工作流执行 |
| 更新工作流 | `n8n_update_partial_workflow` | 差异更新工作流 |
| 版本管理 | `n8n_workflow_versions` | 版本历史和回滚 |
| 自动修复 | `n8n_autofix_workflow` | 自动修复常见问题 |

## 工作流程

### 阶段 1：需求澄清

收到用户需求后，先确认以下信息：

1. **触发方式**：手动 / 定时 / Webhook / 其他事件触发
2. **输入源**：API / 数据库 / 文件 / 表单
3. **处理逻辑**：数据转换 / AI 处理 / 条件判断
4. **输出目标**：通知 / 存储 / API 调用
5. **错误处理**：失败重试 / 告警通知

### 阶段 2：节点选择

**优先顺序：**

1. 搜索模板 → 看是否有现成方案
2. 搜索节点 → 找到合适的节点类型
3. 获取节点文档 → 了解配置方式
4. 参考 n8n-skills → 查看详细示例

**常用节点速查：**

| 场景 | 推荐节点 |
|------|----------|
| HTTP 请求 | `nodes-base.httpRequest` |
| 定时触发 | `nodes-base.scheduleTrigger` |
| Webhook | `nodes-base.webhook` |
| 数据转换 | `nodes-base.set`, `nodes-base.code` |
| 条件判断 | `nodes-base.if`, `nodes-base.switch` |
| AI 处理 | `nodes-langchain.agent`, `nodes-base.openAi` |

### 阶段 3：工作流设计

**步骤：**

1. 用 `get_node` 获取每个节点的配置要求
2. 设计节点连接关系
3. 用 `validate_node` 验证每个节点配置
4. 用 `validate_workflow` 验证完整工作流

**关键原则：**

> ⚠️ **永远不要信任默认值** - 默认参数是运行时失败的首要原因，必须显式配置所有参数。

### 阶段 4：部署和测试

**需要配置 n8n API Key：**

1. `n8n_create_workflow` - 创建工作流
2. `n8n_test_workflow` - 测试执行
3. 检查执行结果
4. 如有问题，用 `n8n_autofix_workflow` 尝试自动修复

## 参考资源

### n8n-skills 知识库

位置：`C:\Users\阿豪\.agents\skills\n8n-skills\`

| 资源 | 路径 | 用途 |
|------|------|------|
| 节点索引 | `resources/INDEX.md` | 按分类查找节点 |
| 使用指南 | `resources/guides/usage-guide.md` | 节点使用方法 |
| 模板参考 | `resources/templates/` | 工作流模板示例 |

### 节点查找技巧

1. **按功能搜索**：`search_nodes({query: "email"})`
2. **按分类浏览**：查看 n8n-skills 的 INDEX.md
3. **带示例搜索**：`search_nodes({query: "slack", includeExamples: true})`

## 安全警告

> ⚠️ **重要提示**
> 
> 1. **不使用删除功能** - `n8n_delete_workflow` 工具不会被使用
> 2. **版本优先** - 修改前先查看版本历史
> 3. **测试先行** - 重大修改先在测试环境验证

## 运行模式

### 全自动模式（默认）

AI 全程处理，仅在以下情况请求用户确认：
- 创建工作流前确认设计
- 执行前确认测试数据
- 优化方案需要评审

### 手动介入模式

用户可在任何阶段要求：
- "让我看看设计方案"
- "暂停，我要检查配置"
- "等我确认再继续"
