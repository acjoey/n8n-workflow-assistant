# 常用工作流模式

## 目录

1. [数据采集模式](#1-数据采集模式)
2. [AI 处理模式](#2-ai-处理模式)
3. [通知分发模式](#3-通知分发模式)
4. [数据同步模式](#4-数据同步模式)
5. [表单处理模式](#5-表单处理模式)
6. [批量处理模式](#6-批量处理模式)

---

## 1. 数据采集模式

### 场景
定时从外部 API 或网站采集数据，存储到数据库或文件。

### 流程
```
Schedule Trigger → HTTP Request → Set (转换) → Database/Sheets
```

### 节点配置要点

| 节点 | 配置 |
|------|------|
| Schedule Trigger | 设置 cron 表达式或间隔时间 |
| HTTP Request | 配置 URL、Headers、认证 |
| Set | 提取和转换需要的字段 |
| Database | 配置连接、表名、操作类型 |

### 示例：定时采集 API 数据到 Google Sheets

```json
{
  "nodes": [
    {
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": {
        "rule": { "interval": [{ "field": "hours", "hoursInterval": 1 }] }
      }
    },
    {
      "type": "n8n-nodes-base.httpRequest",
      "parameters": {
        "url": "https://api.example.com/data",
        "method": "GET",
        "headers": { "Authorization": "Bearer {{$credentials.apiToken}}" }
      }
    },
    {
      "type": "n8n-nodes-base.googleSheets",
      "parameters": {
        "operation": "appendOrUpdate",
        "sheetId": "your-sheet-id"
      }
    }
  ]
}
```

---

## 2. AI 处理模式

### 场景
使用 AI 处理文本、图片或数据，如分类、摘要、翻译等。

### 流程
```
Trigger → Get Data → AI Node → Process Result → Output
```

### 常用 AI 节点

| 节点 | 用途 |
|------|------|
| `openAi` | GPT 文本生成、对话 |
| `agent` | AI Agent 复杂任务 |
| `textClassifier` | 文本分类 |
| `sentimentAnalysis` | 情感分析 |
| `informationExtractor` | 信息提取 |

### 示例：评论自动分类

```json
{
  "nodes": [
    {
      "type": "n8n-nodes-base.webhook",
      "parameters": { "path": "review-classifier" }
    },
    {
      "type": "@n8n/n8n-nodes-langchain.openAi",
      "parameters": {
        "resource": "text",
        "operation": "message",
        "prompt": "将以下评论分类为：正面、负面、中性。只返回分类结果。\n\n评论：{{$json.review}}"
      }
    },
    {
      "type": "n8n-nodes-base.set",
      "parameters": {
        "values": {
          "review": "={{$json.review}}",
          "category": "={{$node['OpenAI'].json.text}}"
        }
      }
    }
  ]
}
```

---

## 3. 通知分发模式

### 场景
监控事件或条件，触发多渠道通知。

### 流程
```
Trigger → Check Condition → IF → [Slack/Email/Telegram...]
```

### 常用通知节点

| 节点 | 渠道 |
|------|------|
| `slack` | Slack 消息 |
| `emailSend` | SMTP 邮件 |
| `telegram` | Telegram 消息 |
| `discord` | Discord 消息 |

### 示例：条件告警通知

```json
{
  "nodes": [
    {
      "type": "n8n-nodes-base.webhook",
      "parameters": { "path": "alert" }
    },
    {
      "type": "n8n-nodes-base.if",
      "parameters": {
        "conditions": {
          "number": [{
            "value1": "={{$json.severity}}",
            "operation": "largerEqual",
            "value2": 3
          }]
        }
      }
    },
    {
      "type": "n8n-nodes-base.slack",
      "parameters": {
        "channel": "#alerts",
        "text": "🚨 告警：{{$json.message}}"
      }
    }
  ]
}
```

---

## 4. 数据同步模式

### 场景
在两个系统之间同步数据，保持一致性。

### 流程
```
Schedule/Webhook → Get Source → Compare → Create/Update Target
```

### 关键节点

| 节点 | 用途 |
|------|------|
| `compareDatasets` | 比较两个数据集差异 |
| `merge` | 合并数据 |
| `removeDuplicates` | 去重 |

### 示例：Google Sheets 同步到数据库

```json
{
  "nodes": [
    {
      "type": "n8n-nodes-base.scheduleTrigger",
      "parameters": { "rule": { "interval": [{ "field": "hours", "hoursInterval": 1 }] } }
    },
    {
      "type": "n8n-nodes-base.googleSheets",
      "parameters": { "operation": "read", "sheetId": "source-sheet" }
    },
    {
      "type": "n8n-nodes-base.postgres",
      "parameters": { "operation": "select", "table": "target_table" }
    },
    {
      "type": "n8n-nodes-base.compareDatasets",
      "parameters": { "mergeByFields": "id" }
    },
    {
      "type": "n8n-nodes-base.postgres",
      "parameters": { "operation": "upsert", "table": "target_table" }
    }
  ]
}
```

---

## 5. 表单处理模式

### 场景
接收表单提交，处理数据，返回结果或发送通知。

### 流程
```
Form Trigger → Validate → Process → Respond/Notify
```

### 示例：表单提交处理

```json
{
  "nodes": [
    {
      "type": "n8n-nodes-base.formTrigger",
      "parameters": {
        "formTitle": "联系表单",
        "formFields": {
          "values": [
            { "fieldLabel": "姓名", "fieldType": "text", "requiredField": true },
            { "fieldLabel": "邮箱", "fieldType": "email", "requiredField": true },
            { "fieldLabel": "留言", "fieldType": "textarea" }
          ]
        }
      }
    },
    {
      "type": "n8n-nodes-base.emailSend",
      "parameters": {
        "fromEmail": "noreply@example.com",
        "toEmail": "support@example.com",
        "subject": "新的联系表单：{{$json.姓名}}",
        "text": "姓名：{{$json.姓名}}\n邮箱：{{$json.邮箱}}\n留言：{{$json.留言}}"
      }
    }
  ]
}
```

---

## 6. 批量处理模式

### 场景
处理大量数据，避免超时或 API 限流。

### 流程
```
Trigger → Get All Data → Split In Batches → Process → Wait → Loop
```

### 关键节点

| 节点 | 用途 |
|------|------|
| `splitInBatches` | 分批处理 |
| `wait` | 等待（避免限流） |
| `loop` | 循环处理 |

### 示例：批量 API 调用

```json
{
  "nodes": [
    {
      "type": "n8n-nodes-base.manualTrigger",
      "parameters": {}
    },
    {
      "type": "n8n-nodes-base.googleSheets",
      "parameters": { "operation": "read" }
    },
    {
      "type": "n8n-nodes-base.splitInBatches",
      "parameters": { "batchSize": 10 }
    },
    {
      "type": "n8n-nodes-base.httpRequest",
      "parameters": { "url": "https://api.example.com/process" }
    },
    {
      "type": "n8n-nodes-base.wait",
      "parameters": { "amount": 1, "unit": "seconds" }
    }
  ]
}
```

---

## 错误处理最佳实践

### 添加错误处理节点

```
Main Flow → Error Trigger → Log Error → Notify
```

### 关键配置

1. **节点级别**：在节点设置中启用 "Continue On Fail"
2. **工作流级别**：添加 Error Trigger 节点捕获全局错误
3. **重试逻辑**：使用 Loop + IF 实现重试

### 错误通知模板

```json
{
  "type": "n8n-nodes-base.slack",
  "parameters": {
    "channel": "#errors",
    "text": "❌ 工作流错误\n工作流：{{$workflow.name}}\n节点：{{$node.name}}\n错误：{{$json.error.message}}"
  }
}
```
