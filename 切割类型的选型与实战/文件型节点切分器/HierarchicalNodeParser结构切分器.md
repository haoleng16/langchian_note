---
date: 2026-01-14
---
# 概述
HierarchicalNodeParser结合文档结构（标题、章节、段落）和语义边界进行多层次切分，适合处理**Markdown、PDF、Word**等结构化文档，尤其适用于说明书、规约、设计文档等场景。其核心优势在于保留文档原生逻辑层级，支持父节点（章节标题+简介）与子节点（具体段落）的嵌套组织。

# 应用场景
**结构文档**（如.md、.pdf、.docx）：默认使用HierarchicalParser，优先保留文档结构
**非结构文档**（如.txt、.csv）：默认使用SentenceSplitter，平衡效率与基础语义完整性

# 代码实现
```
import re
import json
from typing import List
from llama_index.core import Document
from llama_index.core.node_parser.relational.hierarchical import HierarchicalNodeParser

#构造示例中文较长文档
text = (
    "第一章：公司发展背景。公司成立于2005年，最初是一家小型软件外包公司。"
    "随着云计算与大数据的兴起，公司在2010年转型为云服务提供商，"
    "并在2015年完成了A轮融资，融资金额达数千万美元。"
    "第二章：产品与服务。公司主要提供数据分析平台、实时流处理系统和人工智能模型服务。"
    "其中，数据分析平台支持海量日志处理；流处理系统可实现毫秒级别延迟。"
    "第三章：市场与竞争。国内外竞争者众多，我们面临来自大型互联网公司的压力，"
    "但我们的优势在于垂直行业深耕与定制化服务。"
    "第四章：未来展望。我们计划在2026年前进入国际市场，并开展亚太地区的业务。"
)

print("=== 切分前（原文） ===")
print(text)
print("\n")

# 创建 HierarchicalNodeParser
node_parser = HierarchicalNodeParser.from_defaults(
    chunk_sizes=[300, 120],   # 例：300字符/120字符级别
    chunk_overlap=30,         # 重叠区域大小
    include_metadata=True,      # 是否包含metadata
    include_prev_next_rel=True  # 是否包含前后关系
)

nodes = node_parser.get_nodes_from_documents([Document(text=text, metadata={"doc_id":"示例文档"})])

print("=== 切分后（nodes） ===")
for idx, node in enumerate(nodes):
    print(f"--- node {idx} ---")
    print("relationships:", node.relationships)
    print("=" * 60)
    print("text:", repr(node.text))
    print("metadata keys:", list(node.metadata.keys()))
    # 可显示 chunk size 或 parent id
    print("metadata sample:", {k: node.metadata.get(k) for k in ["chunk_size","chunk_level","doc_id"] if k in node.metadata})
    print()
```

## 参数解析

1 chunk_sizes=[300, 120] (核心参数)
- **300 (父节点/Parent)**：系统首先会把文档按 300 字符切分。这些是大块，包含了较完整的上下文。
- **120 (子节点/Child)**：系统会在每个 300 字符的父节点内部，再将其细切成多个 120 字符的小块。
- **作用**：检索时，我们用 **120** 字符的小块去做匹配（因为切得越细，语义越聚焦，匹配越准）；匹配到之后，系统可以通过关系链找到它对应的 **300** 字符的“爸爸”，把“爸爸”的内容喂给 LLM，从而提供更多上下文。

2. chunk_overlap=30
	**含义**：切分时的重叠区域大小
	**作用**：防止关键信息刚好在切分刀口上被切断，导致语义丢失
	
3. include_metadata=True
	**含义**：是否保留元数据。
	**作用**：RAG 必备，方便后续做引用来源追踪
	
4. include_prev_next_rel=True
	**含义**：是否记录前后节点关系
	**作用**：虽然层级索引主要靠“父子”关系，但有了“兄弟”关系，可以让检索器在必要时横向扩展阅读范围