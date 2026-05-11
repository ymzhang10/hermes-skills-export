# Official Document Format Refresh — word-doc Variant

Source: archived skill `official-document-format-refresh-word-doc`

## When to Use THIS Variant

Use when: "把原文内容抽出来，重新排成规范机关公文 docx"

Use the standard `official-document-format-refresh` skill when: preserving original docx structure, annotations, tracked changes, images, header/footer XML detail.

## Execution Dependency

Load `word-doc` skill first — it provides the `.doc/.docx` read, `python-docx` generation, Chinese font and paragraph format control.

This skill adds: 机关公文语义识别, 段落分型, 规范版式映射, 交付核对.

## Paragraph Classification (段落分型)

| 段落类型 | 标识方式 | 格式规则 |
|---------|---------|---------|
| 标题 (Title) | 居中，字号最大 | 二号方正小标宋，居中 |
| 主送机关 | 顶格，冒号结尾 | 三号仿宋，顶格 |
| 正文 | 首行缩进2字符 | 三号仿宋，行距28磅 |
| 附件标注 | "附件：" 开头 | 三号仿宋 |
| 落款机关 | 右对齐 | 三号仿宋 |
| 成文日期 | 右对齐，年月日 | 三号仿宋，汉字数字 |

## python-docx Generation Workflow

```python
from docx import Document
from docx.shared import Pt, Cm
from docx.enum.text import WD_ALIGN_PARAGRAPH

doc = Document()
# Page setup: A4, margins per GB/T 9704-2012
section = doc.sections[0]
section.page_width = Cm(21)
section.page_height = Cm(29.7)
section.top_margin = Cm(3.7)   # 37mm
section.bottom_margin = Cm(3.5)
section.left_margin = Cm(2.8)
section.right_margin = Cm(2.6)

# Title
p = doc.add_paragraph()
p.alignment = WD_ALIGN_PARAGRAPH.CENTER
run = p.add_run("标题文字")
run.font.name = "方正小标宋简体"
run.font.size = Pt(22)  # 二号
```

## Delivery Checklist

- [ ] Title in 方正小标宋 二号, centered
- [ ] Body in 仿宋_GB2312 三号, 28pt line spacing
- [ ] 主送机关 at top left with full-width colon (：)
- [ ] Attachment list follows 正文
- [ ] Issuing org + date right-aligned
- [ ] No extra blank lines between sections
- [ ] Date uses Chinese characters: 二○二五年一月一日
