# n8n 实用工作流示例

这个文档提供了使用n8n创建的实用工作流示例，帮助你了解如何解决实际的自动化需求。

## 什么是n8n？

[n8n](https://n8n.io/) 是一个强大的工作流自动化平台，它允许你以可视化的方式连接各种服务和API，创建自动化工作流。n8n提供了400多个预构建的集成，支持JavaScript/Python代码编写，以及AI功能。

## 示例1：自动化内容发布工作流

这个工作流将自动从RSS源获取最新文章，使用OpenAI生成摘要，并将内容发布到社交媒体平台。

### 工作流结构

```
定时触发器(每天上午9点)
     ↓
RSS读取器(获取最新5篇文章)
     ↓
循环(遍历每篇文章)
  ↓
OpenAI节点(生成文章摘要和社交媒体文案)
  ↓
IF节点(检查是否已发布过这篇文章)
  ├─ 是 → 结束当前迭代
  └─ 否 → Twitter节点(发布推文)
           ↓
         LinkedIn节点(发布到LinkedIn)
           ↓
         数据库节点(记录已发布的文章)
```

### 设置步骤

1. **创建新工作流**
   - 登录n8n实例，点击"创建新工作流"
   - 命名为"内容自动发布工作流"

2. **添加定时触发器**
   - 拖入"Schedule"节点
   - 设置为每天上午9点触发
   - 配置时区

3. **配置RSS读取器**
   - 拖入"RSS Read"节点
   - 在URL字段输入目标RSS源
   - 设置"Max Items"为5
   - 设置"Only Return New Items"为true

4. **添加循环节点**
   - 拖入"Split In Batches"节点进行循环
   - 连接RSS节点的输出
   - 配置"Batch Size"为1

5. **设置OpenAI节点**
   - 拖入"OpenAI"节点
   - 连接API密钥
   - 选择"Create Chat Completion"操作
   - 在提示中使用以下模板：
     ```
     您是一位社交媒体专家。请为以下文章创建一个简短的摘要（不超过80字）和一个引人注目的社交媒体帖子（不超过200字）：
     
     标题: {{$node["Split In Batches"].json["title"]}}
     内容: {{$node["Split In Batches"].json["content"]}}
     链接: {{$node["Split In Batches"].json["link"]}}
     
     返回格式:
     {
       "summary": "简短摘要",
       "social_post": "社交媒体帖子内容，包含链接"
     }
     ```

6. **添加条件检查**
   - 拖入"IF"节点
   - 配置条件检查数据库中是否已存在该文章链接

7. **配置社交媒体发布**
   - 拖入"Twitter"节点
   - 连接Twitter账户
   - 设置推文内容为`{{$node["OpenAI"].json["social_post"]}}`
   - 拖入"LinkedIn"节点
   - 连接LinkedIn账户
   - 类似配置

8. **记录已发布文章**
   - 拖入数据库节点(如"MongoDB")
   - 配置存储已发布文章的URL和时间

9. **激活工作流**
   - 保存并激活工作流

## 示例2：自动化客户数据处理

这个工作流将监控共享文件夹中的新客户数据表格，处理数据，并将其导入到CRM系统中。

### 工作流结构

```
文件夹触发器(监控新Excel文件)
       ↓
读取Excel文件
       ↓
数据转换(清理和格式化数据)
       ↓
重复检测(检查CRM中是否存在)
  ├─ 存在 → 更新CRM记录
  └─ 不存在 → 创建新CRM记录
       ↓
发送操作结果邮件通知
```

### 设置步骤

1. **创建新工作流**
   - 命名为"客户数据自动处理"

2. **添加文件夹触发器**
   - 拖入"Folder"节点
   - 配置监控目录路径
   - 设置文件匹配模式为`*.xlsx`

3. **添加Excel读取器**
   - 拖入"Spreadsheet File"节点
   - 选择"Read from file"操作
   - 配置Sheet名称和范围

4. **添加Function节点进行数据清理**
   - 拖入"Function"节点
   - 编写JavaScript函数清理数据:
     ```javascript
     return items.map(item => {
       // 数据清理逻辑
       item.json.email = item.json.email.toLowerCase().trim();
       item.json.name = item.json.name.trim();
       item.json.phone = item.json.phone.replace(/[^0-9+]/g, '');
       return item;
     });
     ```

5. **添加CRM集成**
   - 拖入CRM节点(如"HubSpot"、"Salesforce")
   - 配置重复检测逻辑
   - 设置更新或创建记录的映射

6. **添加邮件通知**
   - 拖入"Email"节点
   - 配置收件人、主题和内容
   - 在内容中包含处理结果摘要

7. **激活工作流**
   - 保存并激活工作流

## 示例3：AI驱动的客户支持工单分类

这个工作流使用人工智能自动分类新的客户支持工单，并将它们分配给适当的团队。

### 工作流结构

```
Webhook触发器(接收新工单)
       ↓
OpenAI节点(分析工单内容)
       ↓
Switch节点(基于分类路由)
  ├─ 技术问题 → 分配给技术支持团队
  ├─ 账单问题 → 分配给财务团队
  ├─ 功能请求 → 分配给产品团队
  └─ 其他 → 分配给客户支持团队
       ↓
数据库更新(记录分类和分配)
       ↓
通知节点(发送通知给相关团队)
```

### 设置步骤

1. **创建新工作流**
   - 命名为"智能工单分类系统"

2. **添加Webhook触发器**
   - 拖入"Webhook"节点
   - 配置为接收POST请求
   - 定义请求体结构

3. **设置OpenAI分析**
   - 拖入"OpenAI"节点
   - 配置以下提示:
     ```
     您是一位专业的客户支持分类专家。请分析以下客户工单，并将其分类为以下类别之一：
     - 技术问题
     - 账单问题
     - 功能请求
     - 其他
     
     工单标题: {{$node["Webhook"].json["title"]}}
     工单内容: {{$node["Webhook"].json["description"]}}
     
     只返回一个类别名称，不要添加任何解释。
     ```

4. **添加Switch节点进行路由**
   - 拖入"Switch"节点
   - 配置条件分支，基于OpenAI返回的分类
   - 为每个分类创建一个输出路径

5. **配置团队分配节点**
   - 为每个路径添加相应的任务分配节点
   - 可以使用HTTP请求、Slack通知或工单系统API

6. **添加数据库记录**
   - 添加数据库节点记录分类结果
   - 存储工单ID、分类和处理时间

7. **设置团队通知**
   - 添加通知节点(如"Slack"、"Email")
   - 配置适当的消息内容和接收者

8. **激活工作流并测试**
   - 保存工作流
   - 使用测试工单验证分类准确性
   - 根据需要调整OpenAI提示

## 高级技巧与最佳实践

### 1. 使用错误处理

在每个关键节点后添加错误处理路径，确保工作流在出现问题时不会完全停止：

```
关键节点
  ↓
IF节点(检查错误)
 ├─ 出错 → 发送错误通知 → 重试或停止
 └─ 正常 → 继续工作流
```

### 2. 使用环境变量

存储API密钥和敏感信息到n8n的环境变量或凭证存储中，而不是直接写在工作流中：

```javascript
// 使用环境变量
const apiKey = $env.MY_API_KEY;
```

### 3. 模块化设计

将复杂工作流拆分为子工作流，使用"Execute Workflow"节点调用：

```
主工作流
  ↓
执行子工作流A
  ↓
执行子工作流B
```

### 4. 添加日志记录

使用"Function"节点在关键点添加日志记录：

```javascript
console.log('处理数据:', items.length, '条记录');
return items;
```

### 5. 定期测试和维护

定期测试工作流并更新依赖项，确保长期可靠运行：

- 创建测试用例验证工作流
- 设置监控检测工作流失败
- 定期审查工作流逻辑和依赖

## 结语

通过这些n8n示例，你可以了解如何利用这个强大的工作流自动化平台解决实际业务问题。n8n的灵活性和广泛的集成使其成为构建复杂自动化流程的理想选择。无论是内容管理、数据处理还是AI驱动的智能工作流，n8n都能满足多样化的自动化需求。

随着你对n8n的深入了解，可以尝试创建更复杂的工作流，利用JavaScript/Python编写自定义逻辑，并将多个服务无缝集成到统一的自动化解决方案中。