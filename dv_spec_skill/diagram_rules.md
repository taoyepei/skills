# 验证环境框图绘制规则

> **说明**：本规则使用 architecture-diagram skill 生成 HTML+SVG 框图，替代原来的 Mermaid 方案。

## 1. Agent独立显示规则

**规则**：每个Agent必须独立显示，不能放在同一个框内

**错误示例**：
```svg
<!-- 错误：将多个Agent放在同一个边界框内 -->
<rect x="100" y="100" width="300" height="150" ... />
<text>SPI Agent</text>
<text>AHB Agent</text>
```

**正确示例**：
```svg
<!-- 正确：每个Agent独立显示 -->
<rect x="100" y="100" width="150" height="60" fill="rgba(8, 51, 68, 0.4)" stroke="#22d3ee"/>
<text>SPI-like Agent</text>

<rect x="100" y="200" width="150" height="60" fill="rgba(6, 78, 59, 0.4)" stroke="#34d399"/>
<text>AHB-Lite Agent</text>
```

## 2. 通过Interface连接

**规则**：Agent与DUT的交互必须通过Interface，连线上需标注Interface名称

**SVG实现**：使用 `<line>` 或 `<path>` 连接，使用 `<text>` 标注Interface名称

```svg
<!-- 双向Interface连接示例 -->
<line x1="240" y1="315" x2="380" y2="315" stroke="#34d399" stroke-width="1.5"
      marker-end="url(#arrowhead-emerald)" marker-start="url(#arrowhead-start-emerald)"/>
<text x="310" y="308" fill="#34d399" font-size="8" text-anchor="middle">AHB-Lite IF</text>
```

## 3. 双向箭头使用规则

**规则**：
- 双向交互（请求+响应）使用双向箭头（marker-start + marker-end）
- 单向交互使用单向箭头（只有 marker-end）

**SVG双向箭头定义**：
```svg
<defs>
  <!-- 终点箭头 -->
  <marker id="arrowhead-emerald" markerWidth="10" markerHeight="7" refX="9" refY="3.5" orient="auto">
    <polygon points="0 0, 10 3.5, 0 7" fill="#34d399"/>
  </marker>
  <!-- 起点箭头 -->
  <marker id="arrowhead-start-emerald" markerWidth="10" markerHeight="7" refX="1" refY="3.5" orient="auto">
    <polygon points="10 0, 0 3.5, 10 7" fill="#34d399"/>
  </marker>
</defs>
```

**双向交互示例**：
- Agent与DUT的请求-响应交互
- 总线Master与Slave的交互

**单向交互示例**：
- Agent到Reference Model的命令传递
- Agent到Coverage Collector的数据传递

## 4. Clock/Reset Generator连接规则

**规则**：
1. Clock/Reset Generator只连接DUT
2. 最后一段连线必须垂直于DUT边界
3. 箭头在DUT边框外侧（不穿过矩形框）

**正确示例**：
```svg
<!-- CRG到DUT：最后一段垂直向下，箭头在DUT顶边外侧 -->
<path d="M 240 137 L 320 137 L 320 170 L 395 170 L 395 268"
      fill="none" stroke="#fbbf24" stroke-width="1.5" marker-end="url(#arrowhead-amber)"/>
<!-- DUT顶边在 y=270，箭头终点在 y=268 -->
```

**错误示例**：
```svg
<!-- 错误：CRG连接到Agent -->
<path d="M 240 137 L 300 137 L 300 315 L 380 315" .../>
<!-- 错误：最后一段水平，箭头不垂直于边界 -->
<path d="M 240 137 L 310 137 L 310 270 L 430 270" .../>
<!-- 错误：箭头进入DUT内部 -->
<path d="M 240 137 L 310 137 L 310 280 L 430 280" .../>
```

## 5. 数据流连线完整性

**必须画出的连线**：

| 源 | 目标 | 连线类型 | 说明 |
|---|---|---|---|
| 输入Agent | DUT | 双向 | 通过Interface双向连接 |
| DUT | 输出Agent | 双向 | 通过Interface双向连接 |
| 输入Agent | Reference Model | 单向 | 命令传递 |
| 输出Agent | Scoreboard | 单向 | 事务传递 |
| Reference Model | Scoreboard | 单向 | 期望值传递 |
| 所有Agent | Coverage Collector | 单向虚线 | 覆盖率数据收集 |

## 6. 连线不穿越矩形框

**规则**：连线必须绕过组件，不能穿越任何矩形框

**实现方法**：
1. 使用折线（path）绕行
2. 在组件之间的空白区域穿过
3. 必要时从边界框外侧绕行

**正确示例**：
```svg
<!-- SPI Agent到Reference Model：从左侧绕过其他组件 -->
<path d="M 400 165 L 400 195 L 65 195 L 65 510 L 80 510"
      fill="none" stroke="#22d3ee" stroke-width="1.5" marker-end="url(#arrowhead-cyan)"/>
```

## 7. 连线起止点位置

**规则**：连接点在矩形边缘中间位置，不靠近角落

**计算方法**：
- 左/右边缘：y = rect_y + rect_height/2
- 顶/底边缘：x = rect_x + rect_width/2

**示例**：
```svg
<!-- DUT: x=382, y=270, width=216, height=90 -->
<!-- 左边缘中心点：(382, 270+45) = (382, 315) -->
<!-- AHB Agent连接到DUT左边缘中心 -->
<line x1="240" y1="315" x2="382" y2="315" .../>
```

## 8. 不同颜色连线不重合

**规则**：不同颜色的垂直连线不要使用相同的x坐标

**错误示例**：
```svg
<!-- CRG→DUT 和 SPI→RM 都在 x=430 垂直下落，重合 -->
<path d="M 240 137 L 310 137 L 310 170 L 430 170 L 430 268" stroke="#fbbf24"/>
<path d="M 430 165 L 430 195 L 65 195 ..." stroke="#22d3ee"/>
```

**正确示例**：
```svg
<!-- CRG→DUT 在 x=395，SPI→RM 在 x=400，保持间隔 -->
<path d="M 240 137 L 320 137 L 320 170 L 395 170 L 395 268" stroke="#fbbf24"/>
<path d="M 400 165 L 400 195 L 65 195 ..." stroke="#22d3ee"/>
```

## 9. Coverage连线使用虚线

**规则**：Coverage Collector的连线使用虚线，与数据流实线区分

**SVG实现**：
```svg
<line x1="180" y1="350" x2="180" y2="430"
      stroke="#fb7185" stroke-width="1.5" stroke-dasharray="4,4"
      marker-end="url(#arrowhead-rose)"/>
```

## 10. DUT尺寸规则

**规则**：DUT尺寸比其他组件大约20%，突出中心位置

**计算示例**：
- 其他组件：width=180, height=60
- DUT：width=216, height=90 (180*1.2, 60*1.5)

## 11. UVM组件SVG颜色映射

基于 architecture-diagram skill 的颜色系统，UVM组件映射如下：

| UVM 组件类型 | architecture-diagram 角色 | Fill (rgba) | Stroke |
|-------------|-------------------------|-------------|--------|
| Stimulus Agent | Frontend | `rgba(8, 51, 68, 0.4)` | `#22d3ee` (cyan) |
| Response Agent | Backend | `rgba(6, 78, 59, 0.4)` | `#34d399` (emerald) |
| DUT | 独立定义 | `rgba(34, 197, 94, 0.2)` | `#22c55e` (green, stroke-width=2) |
| CRG | Cloud/基础设施 | `rgba(120, 53, 15, 0.3)` | `#fbbf24` (amber) |
| Reference Model | Backend | `rgba(6, 78, 59, 0.4)` | `#34d399` (emerald) |
| Scoreboard | Message Bus | `rgba(251, 146, 60, 0.3)` | `#fb923c` (orange) |
| Coverage Collector | Security | `rgba(136, 19, 55, 0.4)` | `#fb7185` (rose) |
| External Interface Agent | Database | `rgba(76, 29, 149, 0.4)` | `#a78bfa` (violet) |
| Testbench Top 边界 | Region | `rgba(30, 41, 59, 0.3)` | `#94a3b8` (slate, dashed) |
| UVM Environment 边界 | Region | `rgba(8, 51, 68, 0.15)` | `#22d3ee` (cyan) |

## 12. 根据LRS接口类型确定Agent名称

| LRS接口类型 | Agent名称 | 颜色类型 |
|---|---|---|
| SPI类接口 | SPI-like Agent | Stimulus (cyan) |
| I2C接口 | I2C Agent | Stimulus (cyan) |
| UART接口 | UART Agent | Stimulus (cyan) |
| AHB总线（Slave） | AHB-Lite Slave Agent | Response (emerald) |
| AHB总线（Master） | AHB-Lite Master Agent | Stimulus (cyan) |
| AXI总线 | AXI Agent | Response (emerald) |
| APB总线 | APB Agent | Response (emerald) |
| 自定义接口 | 按接口功能命名 | 根据方向确定 |

## 13. 框图布局结构

标准UVM验证环境框图布局：

```
+------------------------------------------+
| Testbench Top                             |
| +----------------------------------------+ |
| | UVM Environment                         | |
| | +----------+  +----------+              | |
| | |   CRG    |  |SPI Agent |              | |
| | +----------+  +----------+              | |
| |       |            |                    | |
| |       v            v                    | |
| | +----------+  +----------+  +----------+| |
| | |AHB Agent |<-|   DUT    |->| CSR Agent|| |
| | +----------+  +----------+  +----------+| |
| |       |            |            |       | |
| |       v            v            v       | |
| | +----------+  +----------+  +----------+| |
| | | Ref Model|->|Scoreboard|<-|Coverage || |
| | +----------+  +----------+  |Collector|| |
| |                             +----------+| |
| +----------------------------------------+ |
+------------------------------------------+
```

## 14. SVG组件绘制顺序

**关键规则**：SVG按文档顺序绘制，后绘制的元素会覆盖先绘制的

**推荐顺序**：
1. 背景 grid
2. 边界框（Testbench Top, UVM Environment）
3. 连线（箭头）
4. 组件矩形框（覆盖在连线上）
5. 文本标签
6. Legend

**遮盖箭头**：由于组件使用半透明填充，箭头可能透过来。如需完全遮盖，先绘制一个不透明背景：
```svg
<rect x="X" y="Y" width="W" height="H" rx="6" fill="#0f172a"/>
<rect x="X" y="Y" width="W" height="H" rx="6" fill="rgba(8, 51, 68, 0.4)" stroke="#22d3ee"/>
```
