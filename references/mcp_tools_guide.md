# n8n-MCP 工具使用指南

## 工具概览

n8n-MCP 提供 19 个工具，分为以下类别：

## 1. 发现工具

### search_nodes

搜索 n8n 节点。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| query | string | 搜索关键词 |
| limit | number | 返回数量（默认 20） |
| mode | string | OR / AND / FUZZY |
| includeExamples | boolean | 是否包含示例 |

**示例**：
```
search_nodes({query: "slack", includeExamples: true})
search_nodes({query: "AI agent langchain"})
search_nodes({query: "trigger webhook"})
```

---

## 2. 配置工具

### get_node

获取节点详细信息。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| nodeType | string | 节点类型（必填） |
| detail | string | minimal / standard / full |
| mode | string | info / docs / search_properties / versions |
| includeExamples | boolean | 是否包含示例 |

**使用策略**：
1. **首次查看**：用 `detail: "standard"` 获取基本信息
2. **需要详情**：用 `detail: "full"` 获取完整 schema
3. **查找属性**：用 `mode: "search_properties"` + `propertyQuery`
4. **看文档**：用 `mode: "docs"` 获取可读文档

**示例**：
```
get_node({nodeType: "nodes-base.slack", detail: "standard"})
get_node({nodeType: "nodes-base.httpRequest", mode: "docs"})
get_node({nodeType: "nodes-base.openAi", mode: "search_properties", propertyQuery: "model"})
```

---

## 3. 验证工具

### validate_node

验证单个节点配置。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| nodeType | string | 节点类型（必填） |
| config | object | 节点配置（必填） |
| mode | string | full / minimal |

**示例**：
```
validate_node({
  nodeType: "nodes-base.slack",
  config: {
    resource: "message",
    operation: "post",
    channel: "#general",
    text: "Hello"
  },
  mode: "full"
})
```

### validate_workflow

验证完整工作流。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| workflow | object | 工作流 JSON（必填） |
| options | object | 验证选项 |

**示例**：
```
validate_workflow({
  workflow: {
    nodes: [...],
    connections: {...}
  }
})
```

---

## 4. 模板工具

### search_templates

搜索工作流模板。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| searchMode | string | keyword / by_nodes / by_task / by_metadata |
| query | string | 搜索关键词（keyword 模式） |
| nodeTypes | array | 节点类型列表（by_nodes 模式） |
| task | string | 任务类型（by_task 模式） |
| complexity | string | simple / medium / complex |
| limit | number | 返回数量 |

**任务类型**：
- ai_automation
- data_sync
- webhook_processing
- email_automation
- slack_integration
- data_transformation
- file_processing
- scheduling
- api_integration
- database_operations

**示例**：
```
search_templates({query: "slack notification"})
search_templates({searchMode: "by_task", task: "ai_automation"})
search_templates({searchMode: "by_nodes", nodeTypes: ["n8n-nodes-base.slack"]})
search_templates({searchMode: "by_metadata", complexity: "simple"})
```

### get_template

获取模板详情。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| templateId | number | 模板 ID（必填） |
| mode | string | nodes_only / structure / full |

**示例**：
```
get_template({templateId: 5531, mode: "full"})
```

---

## 5. n8n API 工具（需要 API Key）

### n8n_create_workflow

创建新工作流。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| name | string | 工作流名称（必填） |
| nodes | array | 节点列表（必填） |
| connections | object | 连接关系（必填） |
| active | boolean | 是否激活 |

### n8n_test_workflow

测试/触发工作流。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| id | string | 工作流 ID（必填） |
| mode | string | webhook / form / chat / execute |
| data | object | 测试数据 |

### n8n_update_partial_workflow

增量更新工作流。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| id | string | 工作流 ID（必填） |
| operations | array | 更新操作列表 |

### n8n_autofix_workflow

自动修复工作流问题。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| id | string | 工作流 ID（必填） |

### n8n_workflow_versions

查看/管理版本历史。

**参数**：
| 参数 | 类型 | 说明 |
|------|------|------|
| id | string | 工作流 ID（必填） |
| action | string | list / get / restore |
| versionId | string | 版本 ID（restore 时需要） |

### n8n_health_check

检查 n8n API 连接状态。

**参数**：无

---

## 最佳实践

### 1. 标准工作流程

```
1. tools_documentation() - 了解工具用法
2. search_templates() - 找现成方案
3. search_nodes() - 找需要的节点
4. get_node(detail: "standard") - 获取配置要求
5. validate_node() - 验证节点配置
6. validate_workflow() - 验证完整工作流
7. n8n_create_workflow() - 创建工作流
8. n8n_test_workflow() - 测试执行
```

### 2. 并行执行

当操作独立时，并行执行以提高效率：

✅ **好的做法**：
```
同时调用 search_nodes(), search_templates()
同时验证多个节点 validate_node()
```

❌ **不好的做法**：
```
依次调用每个工具，等待完成后再调用下一个
```

### 3. 永不信任默认值

⚠️ 默认参数是运行时失败的首要原因。

```json
// ❌ 会失败
{"resource": "message", "operation": "post", "text": "Hello"}

// ✅ 会成功
{
  "resource": "message",
  "operation": "post",
  "channel": "#general",
  "text": "Hello",
  "authentication": "oAuth2"
}
```

### 4. 模板优先

总是先搜索模板，2700+ 模板覆盖大部分场景：

```
search_templates({query: "your use case"})
```
