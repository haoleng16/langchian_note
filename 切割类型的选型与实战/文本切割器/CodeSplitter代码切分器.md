---
date: 2026-01-14
---
# 概述
CodeSplitter专为编程语言源代码设计，利用编程语言的抽象语法树（AST）来理解代码结构，确保将代码按功能单元进行分割。

# 核心特性

- 基于抽象语法树（AST）的结构化解析
- 语言特定，需要指定编程语言
- 保持代码块的功能完整性

# 关键参数
1. language='python' ->==指定编程语言==
2. chunk_lines=10 ->每块大约行数
3. chunk_lines_overlap=2   ->块之间重叠行数
4. max_chars=600  ->每块最大字符数

```
from llama_index.core.node_parser import CodeSplitter
from llama_index.core.schema import Document

# 示例Python代码：一个简单的函数和类
sample_code = '''
def calculate_fibonacci(n):
    """计算斐波那契数列的第n项"""
    if n <= 1:
        return n
    else:
        return calculate_fibonacci(n-1) + calculate_fibonacci(n-2)

class MathOperations:
    """一个简单的数学操作类"""

    def __init__(self):
        self.version = "1.0"

    def factorial(self, n):
        """计算阶乘"""
        if n == 0:
            return 1
        result = 1
        for i in range(1, n+1):
            result *= i
        return result

    def is_prime(self, n):
        """判断是否为质数"""
        if n < 2:
            return False
        for i in range(2, int(n**0.5)+1):
            if n % i == 0:
                return False
        return True
'''

# 创建文档对象
# document = Document(text=sample_code)
#
# 初始化CodeSplitter，指定Python语言
code_splitter = CodeSplitter(
    language="python",    # 指定编程语言
    chunk_lines=10,       # 每块大约行数
    chunk_lines_overlap=2, # 块之间重叠行数
    max_chars=600        # 每块最大字符数
)

# 执行切分
# nodes = code_splitter.get_nodes_from_documents([document])
nodes = code_splitter.split_text(sample_code)

print(f"原始代码字符数: {len(sample_code)}")
print(f"切分后的节点数量: {len(nodes)}")
print("\n" + "="*50 + "\n")

# 显示切分结果
for i, node in enumerate(nodes):
    print(f"节点 {i+1} (字符数: {len(node)}):")
    print("-" * 30)
    print(node)
    print("\n" + "="*50 + "\n")
```
