---
date: 2026-01-12
---
# 初始化语言和模型和向量模型
```
def init_llm_model():
    """
    初始化语言模型
    :return:模型
    
    """

    model=init_chat_model(
        model=os.getenv("DEEPSEEK_CHAT_MODEL"),
        model_provider="deepseek",
    )
    print(f"✅ 语言模型加载成功!")

    return model

def init_vector_store():
    """
    初始化向量模型
    """
    embedding = HuggingFaceEmbeddings(
        model_name = "BAAI/bge-large-zh-v1.5",                  # 模型名称
        # cache_folder="../models"
        model_kwargs = {"device": "cpu"},                       # 设备
        encode_kwargs = {"normalize_embeddings": True}          # 编码参数
    )
    print(f"✅ 嵌入模型加载成功!")
    return embedding

#加载ollama本地向量模型bge-m3:latest 
def init_ollama_vector_store():
    """
    初始化ollama本地向量模型
    """
    embedding = OllamaEmbeddings(
    model=os.getenv("OLLAMA_EMBEDDING_MODEL"),  # 明确指定你的 bge-m3:latest 模型
    base_url="http://172.19.48.1:11434",  # Ollama 默认本地地址，可省略
    # 可选：添加其他嵌入相关配置（如温度系数，bge-m3 作为嵌入模型通常无需调整）
    )
    print(f"✅ ollama本地向量模型加载成功!")
    return embedding
```

# 加载文档(txt格式)
```
def load_documents(file_path:str)->list[Document]:
    """
    加载文档
    :param file_path: 文件路径
    :return: 文档列表
    """
    #加载txt文档
    loader = TextLoader(file_path,encoding="utf-8")
    documents = loader.load()
    return documents
```

# 分割文档
```
def split_documents(documents:str)->list[Document]:
    """
    文档切片
    :param documents:文档列表
    :return:文档列表
    """
    text_splitter=RecursiveCharacterTextSplitter(
        chunk_size=100,
        chunk_overlap=5,
    )
    chunks = text_splitter.split_documents(documents)
    return chunks
```

# 构建向量数据库
```
def build_vector_store(chunks:list[Document]):
    """
    构建向量数据库
    :param chunks:文档切片
    :return:向量数据库
    """
    vector_store = FAISS.from_documents(chunks,embedding)
    #保存向量数据库
    vector_store.save_local("../faiss/faiss_simple")
    print(f"✅ 向量数据库构建成功!")
    return vector_store
```
# 从向量数据库检索内容
```
def get_retriever():
    """
    构建检索器
    :return:检索器
    """
    # #获取向量数据库
    vecstor_store=FAISS.load_local(
        "../faiss/faiss_simple", 
        embedding,
        allow_dangerous_deserialization=True,
    )
    #创建检索器
    retriever=vecstor_store.as_retriever(
        search_kwargs={"k":3},
        search_type="similarity",
    )
    print(f"✅ 检索器构建成功!")
    return retriever

def build_rag_chain(query:str,llm):
    """
    构建RAG链
    :param query:查询
    :param llm:语言模型
    :return:RAG链
    """
    #1.格式化文档辅助函数
    def format_docs(docs):
        """
        格式化文档
        :param docs:文档列表
        :return:格式化后的文档
        """
        return "\n\n".join([doc.page_content for doc in docs])
    
    #2获取检索器
    retriever=get_retriever()
    #增强提示词
    template="""你是一个专业的问答助手。请根据以下提供的上下文信息来回答用户的问题。
    如果上下文中没有相关信息，请诚实地告诉用户你不知道，不要编造答案。
    上下文信息：
    {context}
    问题: {query}回答:
    """
    prompt=ChatPromptTemplate.from_template(template)
    #3构建RAG链
    rag_chain={
        "context":retriever|format_docs,"query":RunnablePassthrough()}|prompt|llm|StrOutputParser()
    #返回RAG链
    return rag_chain
```

# 测试使用
```
if __name__ == "__main__":
    #1. 初始化语言模型
    model = init_llm_model()
    #2. 初始化向量模型
    embedding = init_ollama_vector_store()
    while True:
        cmd=input("请输入1|2|3")
        if cmd=="exit":
            break
        if cmd=="1":

            #3. 加载文档
            documents = load_documents(r"/home/helin/workdir/llm_loacal_development/RAG搭建/sample_document.txt")
            #文档分割
            chunks = split_documents(documents)
            #构建向量数据库
            vector_store = build_vector_store(chunks)
        if cmd=="2":
            #2的步骤是从本地知识库中检索知识
            retriever=get_retriever()
            #检索
            results=retriever.invoke("LangChain是什么")
            print(results)
        if cmd=="3":
            while True:
                query=input("请输入你的问题(输入exit退出):")
                if query=="exit":
                    break
                #创建RAG链
                RAG_chain=build_rag_chain(query,model)
                #执行RAG链
                results=RAG_chain.invoke(query)
                print("大模型响应:",results)   
```