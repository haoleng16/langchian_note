---
date: 2026-01-12
---
# 使用FAISS的构建向量数据库和检索
```
#=======================================构建向量数据库========================================
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

#============================================构建检索器=============================================
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
```

# 使用Chroma构建向量数据库和检索
```
#构建向量数据库
#===================================================构建向量数据库========================================
def build_vector_store(chunks:list[Document]):
    """
    构建向量数据库
    :param chunks:文档切片
    :return:向量数据库
    """
    vector_store = Chroma.from_documents(chunks,embedding,persist_directory=PERSIST_PATH )
    #保存向量数据库
    print(f"✅ Chroma向量数据库构建成功!")
    return vector_store

#===================================================构建检索器=============================================
def get_retriever():
    """
    构建检索器
    :return:检索器
    """
    # #获取向量数据库
    vecstor_store=Chroma(
        persist_directory=PERSIST_PATH, 
        embedding_function=embedding,
    )
    #创建检索器
    retriever=vecstor_store.as_retriever(
        search_kwargs={"k":3},
        search_type="similarity",
    )
    print(f"✅ Chroma检索器构建成功!")
    return retriever
```

# 不同点

| 数据库   | FAISS                                                                   | Chroma                                                             |
| ----- | ----------------------------------------------------------------------- | ------------------------------------------------------------------ |
| 保存操作  | 使用save_local(填写路径)                                                      | 直接from_documents(添加关键字参数persist_directory指定创建的数据库路径)               |
| 加载数据库 | 使用==load_local==加载向量数据库并由独有的处理序列化和反序列的方法allow_dangerous_deserialization | Chroma(==persist_directory===路径,==embedding_fuction===embedding模型) |
