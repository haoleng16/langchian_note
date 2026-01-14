---
date: 2026-01-14
---
# 概述
SentenceSplitter是一种基于自然语言句子和段落边界进行分割的解析器，类似于LangChain的RecursiveCharacterTextSplitter。它优先在句子结束处或段落分隔符处进行分割，尽量避免在句子中间切断，以保持语义单元的完整性。

**核心特性与工作原理**
- **保持句子完整性**：首先尝试按句子边界（如中文的"。""！""？"或英文的".""""!"）进行分割

- **递归分割策略**：如果单个句子长度超过chunk_size，递归地使用更小的分隔符进行分割

- **重叠控制**：通过chunk_overlap参数，允许相邻文本块之间有少量重叠的token

- **多语言支持**：通过separator参数可以自定义分隔符


# 关键参数
1. chunk_size:每个文本块的目标最大token数
2. chunk_overlap:相邻文本块之间的重叠的token数
3. separator:用于分割的主要分隔符
4. paragraph_separator:用于识别段落的分隔符


# 代码实现
```
from llama_index.core.node_parser import SentenceSplitter
# 初始化 SentenceSplitter
sentence_splitter = SentenceSplitter(
    chunk_size=512,      # 这里 chunk_size 表示 token 近似或字符近似，视版本调整
    chunk_overlap=64
)
# 将 documents 转成 nodes
# nodes_from_sentences = sentence_splitter.get_nodes_from_documents(documents)
# print(nodes_from_sentences[0].get_content())

# 测试文本
test_text = """在检索增强生成（RAG）系统中，文档切分与 Node 转换作为连接原始数据与语言模型的关键预处理环节，直接决定了系统的检索精度、生成质量及整体性能。行业实践数据表明，90% 的 RAG 效果问题源于元数据与分块策略不当，而通过优化分块策略可使检索准确率提升 30 - 50%，语义分块较固定分块的准确率优势可达 27%。这一技术环节的重要性体现在：分块过大易引入冗余噪音，增加语言模型理解负担；分块过小或切分不当则可能破坏语义连贯性，导致完整知识点被拆分；未能适配文档结构的机械分块方式还会忽视标题、列表等结构化信息，影响信息提取完整性。
LlamaIndex 作为连接自定义数据与大语言模型（LLMs）的核心框架，通过将文档（如 PDF、文本文件）分解为包含文本内容、向量嵌入和元数据的 Node 组件，构建了结构化文档管理的技术范式。其核心抽象在于将原始文档转换为语义连贯的 Node 集合，向量存储仅保留 Node 内容的嵌入向量与文本信息，这一机制简化了索引构建流程并提升了检索相关性。文档切分与 Node 转换的质量不仅影响向量检索的效率，更决定了上下文增强（Context Augmentation）这一 RAG 核心能力的实现效果。
本文聚焦文档切分与 Node 转换的技术实践，结合 LlamaIndex 框架的实现机制，系统调研分块策略设计、元数据管理及 Node 组件化等关键技术点。通过分析行业最佳实践与典型案例，旨在为 RAG 系统开发者提供可落地的优化方案，解决分块噪音、语义断裂、结构信息丢失等核心痛点，为构建高性能检索增强生成应用奠定技术基础。"""


# 按照文本来进行切分
nodes_from_sentences = sentence_splitter.split_text(test_text)
print( "切分后的文本数量: " + str(len(nodes_from_sentences)))
print("===============================================")
print("第一个文本:")
print(nodes_from_sentences[0])
print("===============================================")
print("第二个文本:")
print(nodes_from_sentences[1])
print("===============================================")
```

