# 节点兼容性检查指南

本指南帮助识别和解决 n8n 工作流中的节点兼容性问题。

## 常见问题

### 问题 1: 节点显示 `?` 标记

**原因：** n8n 无法识别节点类型

**可能原因：**
1. 节点 `type` 名称格式错误
2. `typeVersion` 与 n8n 版本不兼容
3. 需要安装社区节点包

### 问题 2: 节点配置报错

**原因：** 节点属性配置不正确

---

## 节点类型名称规范

### 核心节点

| 分类 | 格式 | 示例 |
|------|------|------|
| 基础节点 | `n8n-nodes-base.xxx` | `n8n-nodes-base.httpRequest` |
| Langchain | `@n8n/n8n-nodes-langchain.xxx` | `@n8n/n8n-nodes-langchain.openAi` |

### 常见错误

| ❌ 错误 | ✅ 正确 |
|---------|---------|
| `nodes-base.httpRequest` | `n8n-nodes-base.httpRequest` |
| `nodes-langchain.openAi` | `@n8n/n8n-nodes-langchain.openAi` |
| `openAi` | `@n8n/n8n-nodes-langchain.openAi` |

---

## typeVersion 参考

### 常用节点最新版本

| 节点 | 推荐 typeVersion |
|------|------------------|
| HTTP Request | `4.4` |
| Set | `3.4` |
| Code | `2` |
| If | `2.2` |
| Switch | `3.2` |
| OpenAI | `2.1` |
| Split In Batches | `3` |
| Aggregate | `1` |
| Spreadsheet File | `2` |
| Manual Trigger | `1` |

### 如何获取正确版本

使用 MCP 工具：

```
get_node({
  nodeType: "nodes-base.httpRequest",
  detail: "standard"
})
```

查看返回结果中的 `version` 字段。

---

## 社区节点检查

### 需要安装的常见社区节点

| 功能 | 包名 | 安装命令 |
|------|------|----------|
| 浏览器自动化 | `n8n-nodes-puppeteer` | `npm install n8n-nodes-puppeteer` |
| 网页抓取 | `n8n-nodes-firecrawl` | `npm install n8n-nodes-firecrawl` |
| DeepSeek AI | `n8n-nodes-deepseek` | `npm install n8n-nodes-deepseek` |
| MCP 集成 | `n8n-nodes-mcp` | `npm install n8n-nodes-mcp` |

### 在 n8n 中安装社区节点

1. 打开 n8n 设置
2. 选择 "Community Nodes"
3. 点击 "Install"
4. 输入包名并安装
5. 重启 n8n 生效

---

## 兼容性检查清单

创建工作流前，检查以下项目：

- [ ] 所有节点 `type` 使用完整格式
- [ ] 所有节点 `typeVersion` 使用最新版本
- [ ] Langchain 节点使用 `@n8n/n8n-nodes-langchain.xxx` 格式
- [ ] 识别并列出需要的社区节点
- [ ] 使用 `validate_workflow` 验证工作流

---

## 问题排查流程

```
节点显示 ? 标记
     │
     ▼
检查 type 名称格式
     │
     ├─ 格式错误 → 修正 type 名称
     │
     ▼
检查 typeVersion
     │
     ├─ 版本过旧 → 更新到最新版本
     │
     ▼
检查是否为社区节点
     │
     ├─ 是 → 安装社区节点包
     │
     ▼
重新导入工作流
```

---

## 验证方法

### 使用 MCP 验证

```javascript
// 验证单个节点
validate_node({
  nodeType: "nodes-base.httpRequest",
  config: { /* 节点配置 */ }
})

// 验证完整工作流
validate_workflow({
  workflow: { /* 工作流 JSON */ }
})
```

### 浏览器验证

导入工作流后，检查：
1. 所有节点显示正确图标
2. 点击节点可以打开配置面板
3. 无红色错误提示
