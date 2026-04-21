# 截图和文档插入规则

> **说明**：本规则定义如何使用 selenium + Chrome 将 HTML+SVG 框图截图为 PNG，并插入到 Word 文档中。

## 1. 整体流程

```
HTML框图生成 → selenium加载 → Chrome截图 → PNG保存 → python-docx插入 → Word文档
```

## 2. selenium + Chrome 截图流程

### 2.1 环境要求

- **selenium**：Python 库，用于控制浏览器
- **Chrome 浏览器**：系统已安装
- **webdriver**：使用系统 Chrome，无需额外下载

### 2.2 截图脚本模板

```python
import os
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options

def capture_html_screenshot(html_path, output_png_path):
    """使用Chrome截取HTML页面截图"""
    chrome_options = Options()
    chrome_options.add_argument('--headless')
    chrome_options.add_argument('--disable-gpu')
    chrome_options.add_argument('--window-size=1400,900')
    chrome_options.add_argument('--hide-scrollbars')

    # 使用系统Chrome
    driver = webdriver.Chrome(options=chrome_options)

    try:
        # 加载HTML文件
        abs_path = os.path.abspath(html_path)
        driver.get(f'file:///{abs_path.replace(os.sep, "/")}')

        # 等待页面加载
        time.sleep(2)

        # 截图
        driver.save_screenshot(output_png_path)
        print(f'截图已保存: {output_png_path}')

    finally:
        driver.quit()
```

### 2.3 参数配置

| 参数 | 默认值 | 说明 |
|-----|-------|------|
| window-size | 1400,900 | 浏览器窗口大小 |
| wait_time | 2秒 | 页面加载等待时间 |
| headless | True | 无界面模式 |

### 2.4 调整窗口大小

根据 SVG viewBox 调整窗口大小：

| SVG viewBox | window-size 建议 |
|------------|-----------------|
| 1000x680 | 1200,800 |
| 1100x650 | 1300,850 |
| 1200x700 | 1400,900 |

## 3. python-docx 插入图片流程

### 3.1 替换文档中的占位内容

当文档中已有占位文本（如 Mermaid 代码块），需要替换为图片：

```python
from docx import Document
from docx.shared import Inches

def replace_diagram_in_docx(docx_path, section_identifier, image_path, output_path):
    """将指定章节的占位内容替换为图片"""
    doc = Document(docx_path)
    
    found_section = False
    paras_to_clear = []
    
    for i, para in enumerate(doc.paragraphs):
        # 找到目标章节
        if section_identifier in para.text:
            found_section = True
            continue
        
        # 找到需要替换的内容（如代码块）
        if found_section:
            text = para.text.strip()
            if text.startswith('```') or text == '```':
                paras_to_clear.append(para)
                # 找到结束标记
                for j in range(i+1, min(i+50, len(doc.paragraphs))):
                    pj = doc.paragraphs[j]
                    pj_text = pj.text.strip()
                    paras_to_clear.append(pj)
                    if pj_text == '```':
                        break
                break
    
    if paras_to_clear:
        # 清除占位内容
        for para in paras_to_clear:
            para.clear()
        
        # 插入图片
        if paras_to_clear[0].text == '':
            run = paras_to_clear[0].add_run()
            run.add_picture(image_path, width=Inches(6.0))
            print(f'已替换为图片')
    
    doc.save(output_path)
```

### 3.2 在新位置插入图片

如果文档是新创建或没有占位内容：

```python
def insert_image_at_paragraph(doc, para_index, image_path, width_inches=6.0):
    """在指定段落位置插入图片"""
    para = doc.paragraphs[para_index]
    run = para.add_run()
    run.add_picture(image_path, width=Inches(width_inches))
```

### 3.3 图片宽度配置

| 文档类型 | 图片宽度建议 |
|---------|------------|
| 验证方案文档 | 6.0 英寸 |
| 技术报告 | 5.5 英寸 |
| 简报 | 4.5 英寸 |

## 4. Word 文档中图片位置策略

### 4.1 按章节标题定位

**策略**：找到章节标题（如"4.1 验证环境示意图"），在其后插入图片

```python
def find_section_paragraph_index(doc, section_title):
    """查找章节标题的段落索引"""
    for i, para in enumerate(doc.paragraphs):
        if section_title in para.text:
            return i
    return -1
```

### 4.2 按内容类型定位

**策略**：找到特定内容（如代码块开始标记），替换整个区域

```python
def find_code_block_range(doc, start_marker="```mermaid"):
    """查找代码块的开始和结束索引"""
    start_idx = -1
    end_idx = -1
    
    for i, para in enumerate(doc.paragraphs):
        if start_marker in para.text:
            start_idx = i
        if start_idx >= 0 and para.text.strip() == '```' and i > start_idx:
            end_idx = i
            break
    
    return start_idx, end_idx
```

## 5. 完整示例脚本

```python
#!/usr/bin/env python3
# -*- coding: utf-8 -*-
"""
截图HTML并更新Word文档
"""

import os
import time
from selenium import webdriver
from selenium.webdriver.chrome.options import Options
from docx import Document
from docx.shared import Inches

def capture_html_screenshot(html_path, output_png_path):
    """使用Chrome截取HTML页面截图"""
    chrome_options = Options()
    chrome_options.add_argument('--headless')
    chrome_options.add_argument('--disable-gpu')
    chrome_options.add_argument('--window-size=1400,900')
    chrome_options.add_argument('--hide-scrollbars')

    driver = webdriver.Chrome(options=chrome_options)

    try:
        abs_path = os.path.abspath(html_path)
        driver.get(f'file:///{abs_path.replace(os.sep, "/")}')
        time.sleep(2)
        driver.save_screenshot(output_png_path)
        print(f'截图已保存: {output_png_path}')
    finally:
        driver.quit()

def update_docx_with_image(docx_path, image_path, output_path, section_title='4.1'):
    """更新Word文档，将指定章节的内容替换为图片"""
    doc = Document(docx_path)

    found_section = False
    para_to_modify = None
    paras_to_delete = []

    for i, para in enumerate(doc.paragraphs):
        if section_title in para.text and '验证环境示意图' in para.text:
            found_section = True
            continue

        if found_section and '```mermaid' in para.text:
            para_to_modify = para
            for j in range(i+1, len(doc.paragraphs)):
                paras_to_delete.append(doc.paragraphs[j])
                if '```' in doc.paragraphs[j].text and '```mermaid' not in doc.paragraphs[j].text:
                    break
            break

    if para_to_modify:
        para_to_modify.clear()
        run = para_to_modify.add_run()
        run.add_picture(image_path, width=Inches(6.5))

        for p in paras_to_delete:
            p._element.getparent().remove(p._element)

        print(f'文档已更新: {output_path}')
    else:
        print('未找到目标章节')

    doc.save(output_path)

if __name__ == '__main__':
    html_path = r'D:\output\diagram.html'
    png_path = r'D:\output\diagram.png'
    docx_path = r'D:\output\dv_spec.docx'
    output_path = r'D:\output\dv_spec.docx'

    # 1. 截图HTML
    capture_html_screenshot(html_path, png_path)

    # 2. 更新Word文档
    update_docx_with_image(docx_path, png_path, output_path)

    print('完成！')
```

## 6. 注意事项

### 6.1 Chrome 必须安装

确保系统已安装 Chrome 浏览器，否则 selenium 无法工作。

### 6.2 HTML 文件路径

HTML 文件使用 `file:///` URL 格式，路径分隔符需要转换为 `/`。

### 6.3 图片文件存在性

插入图片前，确保 PNG 文件已成功生成。

### 6.4 文档格式保护

替换内容时，注意保持文档的其他格式（字体、段落间距等）不变。

## 7. 错误处理

### 7.1 Chrome 启动失败

```python
try:
    driver = webdriver.Chrome(options=chrome_options)
except Exception as e:
    print(f'Chrome启动失败: {e}')
    # 备选方案：使用Mermaid
```

### 7.2 图片插入失败

```python
try:
    run.add_picture(image_path, width=Inches(6.0))
except FileNotFoundError:
    print(f'图片文件不存在: {image_path}')
except Exception as e:
    print(f'图片插入失败: {e}')
```