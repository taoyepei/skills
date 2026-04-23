# 数据流描述规则

## 1. 根据LRS接口类型确定Agent

**判断规则**：

| LRS接口特征 | Agent类型 |
|---|---|
| 类SPI/I2C/UART等串行接口 | 对应的串行接口Agent |
| AHB/AXI/APB等总线接口 | 对应的总线Slave/Master Agent |
| 自定义协议接口 | 自定义接口Agent |

## 2. 各组件数据流描述模板

### 输入接口Agent（如SPI-like Agent）
```
{Agent名称}：驱动DUT的{接口名称}接口（方向：{单向/双向}），{响应处理方式}
```

**示例**：
- SPI-like Agent：驱动DUT的类SPI接口（方向：双向），SPI响应在testcase中直接校验

### 输出/总线接口Agent（如AHB-Lite Slave Agent）
```
{Agent名称}：响应DUT的{总线类型}请求（方向：双向），Monitor监控事务发送Scoreboard
```

**示例**：
- AHB-Lite Slave Agent：响应DUT的AHB Master请求（方向：双向），Monitor监控AHB事务发送Scoreboard

### Reference Model
```
Reference Model：输入来自{输入Agent}的命令，输出期望的{输出类型}到Scoreboard
```

**示例**：
- 输入来自SPI-like Agent解析的命令，输出期望的AHB行为到Scoreboard

### Scoreboard
```
Scoreboard：比对{数据来源1}与{数据来源2}
```

**示例**：
- 比对AHB Agent的实际AHB事务与Ref Model的期望AHB行为

**重要**：明确说明比对哪两路数据！

### Coverage Collector
```
Coverage Collector：从{所有Agent列表}收集transaction，收集功能覆盖率
```

**示例**：
- 从SPI-like Agent和AHB-Lite Agent两个Agent收集transaction

**重要**：必须列出所有数据来源！

## 3. 必须说明的事项

### 关于响应处理方式
- 如果响应在testcase中校验，必须说明
- 如果响应在Scoreboard中比对，必须说明比对哪两路数据

### 关于数据来源
- Coverage Collector的数据来源必须完整列出
- 不能只写从一个Agent收集

## 4. 完整数据流描述示例

```
环境工作机制和数据流：

1. SPI-like Agent：驱动DUT的类SPI接口（双向：发送命令+接收响应），SPI响应在testcase中直接校验
2. AHB-Lite Slave Agent：响应DUT的AHB Master请求（双向：接收AHB请求+返回响应），Monitor监控AHB事务发送给Scoreboard
3. Reference Model：输入来自SPI-like Agent解析的命令，输出期望的AHB行为到Scoreboard
4. Scoreboard：比对AHB Agent的实际AHB事务与Ref Model的期望AHB行为
5. Coverage Collector：从SPI-like Agent和AHB-Lite Agent两个Agent收集transaction，收集功能覆盖率

注意：SPI响应的比对不使用Scoreboard，直接在testcase中校验
```
