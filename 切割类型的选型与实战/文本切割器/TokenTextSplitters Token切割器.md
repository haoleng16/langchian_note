---
date: 2026-01-14
---
# 应用场景
TokenTextSplittersToekn专注于将任意文本字符串拆分成多个片段，按字符/句子/token/自定义分隔规则切分，通常只关心文本长度与重叠上下文，不会理解文件格式。

# 核心特性

- 基于token而非字符进行切分，更准确地反映模型处理能力
- 支持不同的token计算方法（如tiktoken、huggingface tokenizer）
- 适用于多语言场景，不同语言的token密度不同

# 示例代码

```
from llama_index.core.node_parser import TokenTextSplitter

# 初始化 TokenTextSplitter
token_splitter = TokenTextSplitter(
    chunk_size=512,        # 每 chunk 目标 token 数（可调）
    chunk_overlap=64,      # 重叠 token 数（可调）
    separator=" "          # 分隔符（一般用空格）
)

# 将 documents 转成 nodes（LlamaIndex 内部 node/fragment）
#nodes_from_tokens = token_splitter.get_nodes_from_documents(documents)
#print(nodes_from_tokens[1].get_content())

# 测试文本
test_text = """在检索增强生成（RAG）系统中，文档切分与 Node 转换作为连接原始数据与语言模型的关键预处理环节，直接决定了系统的检索精度、生成质量及整体性能。行业实践数据表明，90% 的 RAG 效果问题源于元数据与分块策略不当，而通过优化分块策略可使检索准确率提升 30 - 50%，语义分块较固定分块的准确率优势可达 27%。这一技术环节的重要性体现在：分块过大易引入冗余噪音，增加语言模型理解负担；分块过小或切分不当则可能破坏语义连贯性，导致完整知识点被拆分；未能适配文档结构的机械分块方式还会忽视标题、列表等结构化信息，影响信息提取完整性。
LlamaIndex 作为连接自定义数据与大语言模型（LLMs）的核心框架，通过将文档（如 PDF、文本文件）分解为包含文本内容、向量嵌入和元数据的 Node 组件，构建了结构化文档管理的技术范式。其核心抽象在于将原始文档转换为语义连贯的 Node 集合，向量存储仅保留 Node 内容的嵌入向量与文本信息，这一机制简化了索引构建流程并提升了检索相关性。文档切分与 Node 转换的质量不仅影响向量检索的效率，更决定了上下文增强（Context Augmentation）这一 RAG 核心能力的实现效果。
本文聚焦文档切分与 Node 转换的技术实践，结合 LlamaIndex 框架的实现机制，系统调研分块策略设计、元数据管理及 Node 组件化等关键技术点。通过分析行业最佳实践与典型案例，旨在为 RAG 系统开发者提供可落地的优化方案，解决分块噪音、语义断裂、结构信息丢失等核心痛点，为构建高性能检索增强生成应用奠定技术基础。"""

# 按照文本来进行切分
nodes_from_tokens = token_splitter.split_text(test_text)
# 打印切分后的文本数量
print( "切分后的文本数量: " + str(len(nodes_from_tokens)))
print("===============================================")
# 打印第一个文本
print("第一个文本:")
print(nodes_from_tokens[0])
print("===============================================")
# 打印第二个文本
print("第二个文本:")
print(nodes_from_tokens[1])
print("===============================================")
```