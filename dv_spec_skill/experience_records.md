# DV SPEC文档更新经验记录

> **说明**：记录实际项目中遇到的DV SPEC文档更新问题和解决方案。

---

## 一、LRS版本更新处理

### 1.1 版本升级场景

当LRS从旧版本升级到新版本时，DV SPEC需要同步更新：

| 更新类型 | 检查内容 |
|---------|---------|
| 新增功能 | 是否有对应的验证策略、覆盖点、测试场景 |
| 接口变化 | Agent的Interface/Transaction定义是否更新 |
| 架构变化 | 设计架构图、数据流图是否需要重新绘制 |
| 新增命令 | 命令类型覆盖、错误处理是否覆盖 |
| 新增状态码 | 错误优先级链、覆盖点是否更新 |

### 1.2 LRS v2.2 更新示例

**新增内容**：
- Burst命令：AHB_WR_BURST(0x22)、AHB_RD_BURST(0x23)
- Lane模式：1/4/8/16-bit（原仅1/4-bit）
- 内部FIFO：SLC_RXFIFO、SLC_TXFIFO
- 新状态码：STS_BAD_BURST(0x80)、STS_BURST_BOUND(0x81)

**DV SPEC更新要点**：
1. 验证环境示意图：Agent需支持16-bit lane
2. Agent实现方案：Transaction新增burst_len、hburst字段
3. Reference Model：新增Burst处理、FIFO状态建模
4. 覆盖率方案：新增Burst/FIFO/Lane覆盖点
5. 测试场景：新增Burst测试场景、边界测试

---

## 二、Agent实现方案完整性检查

### 2.1 必须包含的四个部分

每个Agent必须包含：

| 部分 | 内容 |
|-----|-----|
| Interface定义 | 信号列表、位宽、clocking block |
| Transaction定义 | 字段名称、位宽、含义 |
| Driver实现方案 | 驱动行为、时序控制、边界条件 |
| Monitor实现方案 | 监控、封装transaction、协议违规检测 |

### 2.2 常见遗漏项

| 遗漏项 | 影响 |
|-------|-----|
| 缺少Monitor实现 | 无法验证协议合规性 |
| 缺少信号位宽 | 接口连接可能出错 |
| 缺少协议违规检测 | 无法发现时序问题 |
| Transaction缺少Burst字段 | Burst测试无法覆盖 |

### 2.3 AHB-Lite Slave Agent 示例

**Interface定义**：
```
信号包括：
• hclk (1-bit) - AHB时钟
• hresetn (1-bit) - AHB复位
• hsel (1-bit) - Slave选择
• haddr (32-bit) - 地址
• hwrite (1-bit) - 写标志
• hsize (3-bit) - 传输大小
• hburst (3-bit) - Burst类型
• htrans (2-bit) - 传输类型
• hwdata (32-bit) - 写数据
• hrdata (32-bit) - 读数据
• hreadyout (1-bit) - Slave就绪
• hresp (1-bit) - 响应状态
支持 clocking block 同步采样
```

**Transaction定义**：
```
• addr：AHB地址（32-bit）
• data：读写数据（32-bit×N，Burst时为数组）
• write：读写标志（1-bit）
• size：传输大小（3-bit）
• burst：Burst类型（3-bit）
• trans：传输类型（2-bit）
• response：响应状态（1-bit）
```

**Driver实现方案**：
```
• 响应DUT的AHB Master请求
• 支持SINGLE和INCR4/8/16 Burst响应
• 支持可配置的hready延迟（反压模拟）
• 支持hresp错误注入
• 支持在Burst任意beat注入错误
```

**Monitor实现方案**：
```
• 监控AHB总线上的读写事务
• 将监控到的事务封装为transaction
• 通过analysis port发送给Scoreboard和Coverage Collector
• 检测AHB协议违规（htrans时序、hready响应、Burst边界）
```

---

## 三、Reference Model架构图绘制

### 3.1 架构图必须展示的内容

| 模块 | 功能说明 |
|-----|---------|
| 命令解析模块 | Opcode解码、参数提取、Lane模式识别 |
| 错误检查模块 | 错误优先级链检查、BAD_BURST、BURST_BOUND |
| Burst处理模块 | 生成N个AHB beat、地址递增、hburst映射 |
| FIFO状态建模 | rxfifo/txfifo状态跟踪、空/满状态 |
| 输出接口 | AHB期望、CSR期望、状态码 |

### 3.2 模块颜色映射

| 模块类型 | Fill (rgba) | Stroke |
|---------|-------------|--------|
| 输入/解析 | rgba(8, 51, 68, 0.4) | #22d3ee (cyan) |
| 错误检查 | rgba(136, 19, 55, 0.3) | #fb7185 (rose) |
| 处理逻辑 | rgba(6, 78, 59, 0.4) | #34d399 (emerald) |
| FIFO建模 | rgba(120, 53, 15, 0.3) | #fbbf24 (amber) |
| 输出接口 | rgba(251, 146, 60, 0.3) | #fb923c (orange) |

### 3.3 命令处理流程图要素

流程图应包含：
1. 命令接收 → Opcode解码
2. Opcode合法性检查 → BAD_OPCODE
3. Lane_mode有效性检查
4. Burst命令识别 → burst_len检查 → BAD_BURST
5. 地址对齐检查
6. 1KB边界检查 → BURST_BOUND
7. 生成AHB期望行为

---

## 四、查漏补缺检查清单

### 4.1 章节完整性检查

| 章节 | 检查项 |
|-----|-------|
| 2.1 设计架构图 | 是否包含新增模块（如FIFO） |
| 2.3 数据流图 | 是否更新为新的段数（如5段） |
| 3.1 验证重点难点 | 是否包含新功能的验证难点 |
| 4.2.1 Agent实现 | 是否完整（Interface/Transaction/Driver/Monitor） |
| 4.2.2 Ref Model | 是否有架构图和流程图 |
| 4.3 覆盖率方案 | 是否包含新功能覆盖点 |
| 6.1 测试场景 | 是否包含新功能测试场景 |

### 4.2 截图完整性检查

| 检查项 | 方法 |
|-------|-----|
| 截图是否全白 | 检查HTML文件是否为空 |
| 边缘是否裁剪 | 打开PNG检查边缘内容 |
| 字体是否加载 | 检查文字是否正确显示 |
| 图表是否完整 | 检查所有组件是否在图中 |

### 4.3 文档格式检查

| 检查项 | 标准 |
|-------|-----|
| 字体 | 宋体正文，黑体标题 |
| 字号 | 正文10.5pt，标题12/14pt |
| 图片宽度 | 450-500像素（适合页面宽度） |
| 表格边框 | 单线边框，黑色 |