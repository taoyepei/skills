# Agent实现方案规则

## 1. 每个Agent必须包含的内容

### Interface定义
- 列出所有信号名称
- 说明位宽
- 说明clocking block（如有）

### Transaction定义
- 列出所有字段名称
- 说明位宽
- 说明含义

### Driver实现方案
- 描述驱动行为
- 说明时序控制
- 说明边界条件处理

### Monitor实现方案
- **必须包含**：监控总线事务
- **必须包含**：将事务封装为transaction发送到analysis port
- **必须包含**：协议违规检测

## 2. Monitor实现方案规范

**必须包含的内容**：

```
Monitor 实现方案：
• 监控{总线类型}总线上的读写事务
• 将监控到的事务封装为transaction
• 通过analysis port发送给Scoreboard和Coverage Collector
• 检测{协议名称}协议违规（如{具体时序检查项}）
```

**示例（AHB）**：
```
Monitor 实现方案：
• 监控AHB总线上的读写事务
• 将监控到的事务封装为transaction
• 通过analysis port发送给Scoreboard和Coverage Collector
• 检测AHB协议违规（如htrans时序、hready响应等）
```

## 3. 只描述数据流中出现的组件

**规则**：如果某个Agent没有在验证环境数据流图中出现，则不应描述其实现方案

**示例**：
- 如果CSR Agent不在数据流图中，不应描述其实现方案
- 如果只有SPI Agent和AHB Agent，只描述这两个

## 4. Agent实现方案模板

```
一、{Agent名称}（自研）

负责驱动和监控{接口名称}接口。

Interface 定义：
信号包括：{信号列表}
支持 clocking block 同步采样

Transaction 定义：
• {字段1}：{描述}（{位宽}）
• {字段2}：{描述}（{位宽}）
...

Driver 实现方案：
• {驱动行为描述}
• {时序控制说明}
...

Monitor 实现方案：
• 监控{总线类型}总线上的读写事务
• 将监控到的事务封装为transaction
• 通过analysis port发送给Scoreboard和Coverage Collector
• 检测{协议名称}协议违规（如{具体检查项}）
```

## 5. 常见错误

| 错误 | 正确做法 |
|---|---|
| 缺少Monitor实现方案 | 必须包含Monitor实现方案 |
| Monitor缺少协议违规检测 | 必须包含协议违规检测 |
| 描述了未在数据流中的Agent | 只描述数据流中出现的Agent |
| Monitor使用"监控"描述比对 | Monitor负责监控，Checker负责比对 |
