# Checker实现规则

## 1. 职责区分

### Monitor的职责
- 监控总线事务
- 将事务封装为transaction
- 发送到analysis port
- **检测协议违规**

### Checker/Scoreboard的职责
- 接收transaction
- **比对数据正确性**
- 报告差异

## 2. 表述规范

**正确表述**：
- "比对AHB总线事务的地址、数据正确性"
- "比对实际输出与期望值"

**错误表述**：
- ~~"监控AHB总线事务，验证地址、数据正确性"~~（监控是Monitor的职责）
- ~~"检测协议违规"~~（这是Monitor的职责）

## 3. Checker实现方案模板

```
比对内容：
• {事务类型}比对：比对{数据内容}的正确性

Checker 架构：
• 使用 UVM scoreboard 组件
• 从{Agent名称}的 analysis port 接收数据
• 从 Reference Model 获取期望值
• 执行比对并报告差异
```

**示例**：
```
比对内容：
• AHB事务比对：比对AHB总线事务的地址、数据正确性

Checker 架构：
• 使用 UVM scoreboard 组件
• 从 AHB Agent 的 analysis port 接收数据
• 从 Reference Model 获取期望值
• 执行比对并报告差异
```

## 4. 数据来源说明

**必须明确**：
- Checker从哪个Agent的analysis port接收数据
- 期望值从哪里获取（通常是Reference Model）

## 5. 与RTM中Checker List的区别

| 验证方案中的Checker实现方案 | RTM中的Checker List |
|---|---|
| 描述Checker如何实现 | 列出有哪些checker |
| 描述比对逻辑 | 不描述实现细节 |
| 描述数据来源 | 只列出检查项 |

## 6. 常见错误

| 错误 | 正确做法 |
|---|---|
| 用"监控"代替"比对" | 用"比对"描述Checker行为 |
| 混淆Monitor和Checker职责 | Monitor监控，Checker比对 |
| 未说明数据来源 | 明确说明从哪个Agent接收数据 |
| 包含"状态码比对" | 状态码比对在testcase中校验，不应在Scoreboard中比对 |

## 7. 比对内容正确性

**规则**：只保留需要Scoreboard比对的内容

**常见错误**：
```
比对内容：
• AHB事务比对：比对AHB总线事务的地址、数据正确性
• 状态码比对：比对命令返回的状态码正确性  ← 错误！状态码应在testcase中校验
```

**正确做法**：
```
比对内容：
• AHB事务比对：比对AHB总线事务的地址、数据正确性

注意：SPI响应（状态码和读数据）在testcase中直接校验，不经过Scoreboard
```

## 8. 比对内容判断规则

| 内容类型 | 是否在Scoreboard比对 |
|---|---|
| 总线事务（AHB/AXI/APB） | ✓ 是 |
| 状态码 | ✗ 否，在testcase中校验 |
| SPI响应数据 | ✗ 否，在testcase中校验 |
| 寄存器访问结果 | 根据架构决定 |
