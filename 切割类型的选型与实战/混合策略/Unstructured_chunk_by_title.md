---
date: 2026-01-14
---
# 概述
核心思想: 利用Title元素作为分段标志,将Title与其后的内容组合成语义完整的chunk
**优势**:
   - 保留文档结构边界
   - 自动合并小段落
   - 留元数据层级信息
   - 避免跨章节混合

# 代码实现
## 使用partition_pdf解析PDF文档为elements类型
```
from unstructured.partition.pdf import partition_pdf

# 使用 partition_pdf 函数解析 PDF 文档 为elements类型
elements = partition_pdf(
    filename="甬兴证券-AI行业点评报告：海外科技巨头持续发力AI，龙头公司中报业绩亮眼.pdf",
    strategy="hi_res",  # 使用高精度模式
    extract_images_in_pdf=False,
)
```
如果想要获取elements里读取到的pdf内容:
elements:是装着多个unstructured库定义的Element对象
```
for element in elements:
    print(element.text)
```
并且设置读取的内容是中文需要
```
from unstructured.partition.pdf import partition_pdf

# 使用 partition_pdf 函数解析 PDF 文档 为elements类型
elements = partition_pdf(
    filename="甬兴证券-AI行业点评报告：海外科技巨头持续发力AI，龙头公司中报业绩亮眼.pdf",
    strategy="hi_res",  # 使用高精度模式
    
    #=========================================添加==============================
    languages=["chi_sim", "eng"],
	    #===========================================================================
    
    extract_images_in_pdf=False,
)
```

## 导入unstructured的chunk_by_title切分方法
**文档切分**
```
from unstructured.chunking.title import chunk_by_title

# 文档切分
chunked_elements = chunk_by_title(
    elements,                       # 读取的元素列表
    max_characters=800,             # 每个chunk的最大字符数
    combine_text_under_n_chars=150, # 小于该字符数的文本块会合并
)

print(f"✓ 解析出 {len(elements)} 个元素")
print(f"✓ 切分成 {len(chunked_elements)} 个chunks")
```
### chunk_by_title的工作逻辑

1. **以标题为边界（Semantic Grouping）：**
    - 当它遇到一个被识别为 Title（标题）的元素时，它倾向于**开启一个新的 Chunk**。
    - 这意味着它会把这个标题以及标题下面的正文（NarrativeText）、列表（ListItem）等内容聚合在一起。
    - **目的**：确保检索（RAG）时，每个块不仅仅是一堆文字，而是“标题+相关内容”，保留了上下文。
2. **遵守长度限制（max_characters=800）：**
    - 它会将元素不断合并到一个 Chunk 中，直到字符总数接近 800。
    - 如果一个段落太长，导致合并后超过 800 字，它会把该段落拆分（Split），多出的部分放入下一个 Chunk。
    
3. **合并碎片文本（combine_text_under_n_chars=150）：**
    - 如果在这个过程中遇到一些很短的独立元素（比如只有 50 个字的短句或孤立的列表项），只要当前的 Chunk 还有空间，它就会强制把这些小段落合并进去，避免产生大量只包含一句话的“碎片 Chunk”。
    - **目的**：避免切分出语义不完整、对模型无意义的微小片段。


### 参数解析

1. elements:原材料的列表
	**作用**：chunk_by_title 会遍历这个列表，利用里面的元数据（比如谁是标题，谁是正文）来决定在哪里切分文本。
2. max_characters=800
	 **含义**：**硬性上限**（每个 Chunk 的最大容量）。
	 **作用**：规定生成的每一个文本块（Chunk）包含的字符数**绝对不能超过** 800 个字符。
3. combine_text_under_n_chars=150
    **含义**：**防碎片化阈值**（小文本合并策略）。
    **作用**：防止生成**太短的、缺乏语义信息的**独立 Chunk。
    
   **为什么需要它？**
    - 在 PDF 解析中，经常会出现一些奇怪的短文本，比如页眉、页脚、或者被错误识别的一句话列表项（例如：“注：数据来源”）。
    - 如果没有这个参数，当一个大的章节结束时，这些遗留的小尾巴可能会被单独做成一个 Chunk。
    - **例子**：假设有一个 Chunk 包含了 10 个字：“数据来源：Wind”。拿这 10 个字去向量库检索没有任何意义，反而会产生噪音。

```
from llama_index.core.schema import TextNode

# 直接转换为Node格式
nodes = []
for i, chunk in enumerate(chunked_elements):
    # 处理metadata
    metadata = chunk.metadata.to_dict()

    # 如果有原始元素,可以提取更多信息
    if hasattr(chunk.metadata, 'orig_elements') and chunk.metadata.orig_elements:
        # 提取所有原始元素的类型
        metadata['orig_element_types'] = [type(e).__name__ for e in chunk.metadata.orig_elements]
        # 标记是否包含Title
        metadata['contains_title'] = any(
            type(e).__name__ == 'Title' for e in chunk.metadata.orig_elements
        )

    # 移除languages
    metadata.pop('languages', None)
    # 移除orig_elements
    metadata.pop('orig_elements', None)

    # 创建TextNode
    node = TextNode(
        text=chunk.text,
        metadata=metadata,
        id_=f"chunk_{i}",
    )
    nodes.append(node)

print(f"✓ 创建 {len(nodes)} 个TextNodes")
print("=" * 80)
print(nodes[0].text)
print("=" * 80)
print(nodes[0].metadata)
```
