# dv_spec_skill

## 描述

根据LRS（Logic Requirement Specification）文档生成高质量的IC验证方案文档。

## 触发条件

当用户提到以下关键词时触发：
- "验证方案"
- "DV spec"
- "验证方案文档"
- "生成验证方案"

## 使用方式

支持两种调用方式：

### 方式1：自然语言调用
```
"调用dv_spec_skill生成验证方案，LRS路径是D:\UART_LRS.docx，输出路径是D:\output\UART_dv_spec.docx"
"用dv_spec_skill帮我生成验证方案文档，输入文件是xxx，输出到xxx"
```

### 方式2：参数化调用
```
dv_spec_skill --lrs "D:\UART_LRS.docx" --output "D:\output\UART_dv_spec.docx"
```

## 工作流程

1. **提取路径参数**：从用户输入中提取LRS路径和输出路径
2. **读取并解析LRS文档**：按章节结构提取模块信息
3. **加载内置验证方案模板**：使用固化的模板结构
4. **按模板填充内容**：用LRS信息填充占位符
5. **绘制验证环境框图**：调用 architecture-diagram skill 生成 HTML+SVG 框图，使用 selenium+Chrome 截图为 PNG，插入到文档中
6. **描述数据流和组件**：编写各组件实现方案
7. **自检错误清单**：逐项检查常见错误
8. **输出验证方案文档**

## 输入

- LRS文档路径（用户提供）
- 输出文档路径（用户提供）

## 输出

验证方案文档（.docx格式）

## 依赖

### 外部技能
本技能依赖以下外部技能：
- **document-skills:docx**：用于读取.docx格式的LRS文档，以及生成.docx格式的验证方案文档
- **architecture-diagram**：用于生成专业深色主题的 HTML+SVG 框图，替代 Mermaid 代码。包括验证环境示意图、设计架构图、流程图等各类技术框图。
- **selenium + Chrome**：用于将 HTML 框图截图为 PNG，插入到 Word 文档中

### 内部规则文件
本技能依赖以下规则文件：
- template.md：内置验证方案模板
- template_understanding.md：模板理解规则
- diagram_rules.md：框图绘制规则（HTML+SVG 方案）
- diagram_screenshot.md：截图和文档插入规则
- dataflow_rules.md：数据流描述规则
- agent_implementation.md：Agent实现规则
- checker_rules.md：Checker实现规则
- coverage_rules.md：覆盖率方案规则
- xprop_rules.md：Xprop配置规则
- mermaid_rules.md：Mermaid流程图规则（备选方案）
- error_checklist.md：常见错误清单
- uvm_diagram_template.html：UVM 验证环境 SVG 模板

## 调用示例

```
用户：调用dv_spec_skill生成验证方案，LRS路径是D:\UART_LRS.docx，输出路径是D:\output\UART_dv_spec.docx

执行流程：
1. 调用 document-skills:docx 读取 LRS 文档
2. 解析 LRS 内容
3. 按规则生成验证方案
4. 调用 document-skills:docx 输出验证方案文档
```

## 核心目标

当用户提供LRS文档路径和输出路径后，能够直接输出与高质量验证方案文档，无需多次修改迭代。
