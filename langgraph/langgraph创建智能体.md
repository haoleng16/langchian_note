---
date: 2026-01-08
---
source:langchain官网<[LangGraph overview - Docs by LangChain](https://docs.langchain.com/oss/python/langgraph/quickstart#full-code-example)>

# 1.定义工具和模型

```
#1. 初始化模型
model = init_chat_model(
    model = os.getenv('DEEPSEEK_CHAT_MODEL'),
    model_provider = 'deepseek',
)

#2. 定义工具
# Define tools
@tool
def multiply(a: int, b: int) -> int:
    """Multiply `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a * b


@tool
def add(a: int, b: int) -> int:
    """Adds `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a + b


@tool
def divide(a: int, b: int) -> float:
    """Divide `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a / b

#3. 工具编排
tools = [multiply, add, divide]
tools_by_name = {tool.name: tool for tool in tools}

#4. 给大模型绑定工具
model_with_tools = model.bind_tools(tools)
```

# 2.定义全局状态类
作用:让state获取状态信息  (包含存储信息和模型调用次数)
```
class MessageState(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]    # 消息列表
    llm_calls: int                                         # 大模型调用次数       
```

# 3.定义模型节点
![[langgraph图.png]]

```
def llm_call_node(state: dict):

    """

    大模型节点

    :param state: 状态

    :return: 状态

    """

    # 获取state

    print(f"消息列表: {state['messages']}")

    print(f"大模型调用次数: {state['llm_calls']}")

  

    #1. 调用大模型

    response = model_with_tools.invoke(

        [

            SystemMessage(content="你是一个AI助手, 你不仅可以与用户进行闲聊, 对于某些用户的问题, 你可以调用工具来回答"),

        ] + state["messages"]

    )

  

    #2. 构建状态

    new_state = {

        "messages": [response],                             # 更新消息列表, 存储的是所有的聊天历史记录

        "llm_calls": state.get("llm_calls", 0) + 1,         # 更新大模型调用次数

    }

    return new_state
    
    
```

# 4. 定义工具节点
```
def tool_node(state: dict):
    """
    工具节点
    :param state: 状态
    :return: 状态
    """
    # result = []
    print("-----------------tool_node-------------------")
    print(f"消息列表: {state['messages']}")
    print(f"大模型调用次数: {state['llm_calls']}")
    print("-----------------tool_node-------------------")
    
    for tool_call in state["messages"][-1].tool_calls:
        #1. 获取工具(根据工具映射表获取工具)
        tool = tools_by_name[tool_call["name"]]

        #2. 调用工具
        tool_result = tool.invoke(tool_call["args"])
        
        #3. 构建一个ToolMessage
        tool_message = ToolMessage(
            content=tool_result,
            tool_call_id=tool_call["id"],
        )
        # result.append(tool_message)
        # return {"messages": result}

        #4. 构建状态
        new_state = {
            "messages": [tool_message],
            "llm_calls": state.get("llm_calls", 0),
        }

        return new_state
```

# 5.定义路由决策函数
如果获取到了ToolMessage则说明有调用工具则进入工具节点否则直接进入end结束
```
def should_continue(state: dict) -> bool:
    """
    路由决策函数
    :param state: 状态
    :return: 是否继续
    """
    print(f"消息列表: {state['messages']}")
    print(f"大模型调用次数: {state['llm_calls']}")

    #1. 获取消息
    messages = state["messages"]

    #2. 获取最近一条消息
    last_message = messages[-1]

    #3. 判断最近一条消息是否是ToolMessage
    if last_message.tool_calls:
        return "tool_node"
    
    return END
    
```

# 6. 构建状态图
```
from langgraph.graph import StateGraph
from langgraph.graph import START, END
#7.1 构建状态图
agent_builder = StateGraph(MessageState) #继承消息状态类
```

# 7.添加节点和边
```
#7.2 添加节点
agent_builder.add_node("llm_call", llm_call_node)
agent_builder.add_node("tool_node", tool_node)

#7.3 添加边
agent_builder.add_edge(START, "llm_call")           # START->llm_call节点
agent_builder.add_conditional_edges(
    "llm_call",
    should_continue,                                # 条件分支,路由决策函数,决策是否调用工具,如果不用调用工具,直接走END节点
    ["tool_node", END]                              # 如果需要调用工具,走tool_node节点,否则走END节点
)
agent_builder.add_edge("tool_node", "llm_call")     # tool_node->llm_call节点
```

# 8.编译显示状态图
```
#7.4 编译状态图
agent = agent_builder.compile()
        
#8. 显示状态图，并保存到一个图片文件中
agent_image = agent.get_graph(xray=True).draw_mermaid_png()
with open("agent_graph.png", "wb") as f:
    f.write(agent_image)
```

# 9.测试使用
```

if __name__ == "__main__":
    while True:
        question = input("请输入问题: ")
        if question == "exit":
            break

        #1. 初始化状态
        state = {
            "messages": [],
            "llm_calls": 0,
        }
        state['messages'].append(HumanMessage(content=question))

        #1. 执行代理
        messages = agent.invoke(input=state)
        # print(messages)
        print(messages['messages'][-1].content, end="", flush=True)
        print()

```

# 10.完整示例
```
from langchain.tools import tool
from langchain.chat_models import init_chat_model
from langchain.messages import SystemMessage, ToolMessage, HumanMessage
from dotenv import load_dotenv
import os

load_dotenv(override=True)

#1. 初始化模型
model = init_chat_model(
    model = os.getenv('DEEPSEEK_CHAT_MODEL'),
    model_provider = 'deepseek',
)

#2. 定义工具
# Define tools
@tool
def multiply(a: int, b: int) -> int:
    """Multiply `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a * b


@tool
def add(a: int, b: int) -> int:
    """Adds `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a + b


@tool
def divide(a: int, b: int) -> float:
    """Divide `a` and `b`.

    Args:
        a: First int
        b: Second int
    """
    return a / b

#3. 工具编排
tools = [multiply, add, divide]
tools_by_name = {tool.name: tool for tool in tools}

#4. 给大模型绑定工具
model_with_tools = model.bind_tools(tools)

#5. 定义状态
from typing import TypedDict, Annotated, Any
import operator
from langchain_core.messages import AnyMessage

class MessageState(TypedDict):
    messages: Annotated[list[AnyMessage], operator.add]    # 消息列表
    llm_calls: int                                         # 大模型调用次数        

#6. 定义节点
#6.1 大模型节点
def llm_call_node(state: dict):
    """
    大模型节点
    :param state: 状态
    :return: 状态
    """
    # 获取state
    print(f"消息列表: {state['messages']}")
    print(f"大模型调用次数: {state['llm_calls']}")

    #1. 调用大模型
    response = model_with_tools.invoke(
        [
            SystemMessage(content="你是一个AI助手, 你不仅可以与用户进行闲聊, 对于某些用户的问题, 你可以调用工具来回答"),
        ] + state["messages"]
    )

    #2. 构建状态
    new_state = {
        "messages": [response],                             # 更新消息列表, 存储的是所有的聊天历史记录
        "llm_calls": state.get("llm_calls", 0) + 1,         # 更新大模型调用次数
    }

    return new_state
     
#6.2 工具节点
def tool_node(state: dict):
    """
    工具节点
    :param state: 状态
    :return: 状态
    """
    # result = []
    print("-----------------tool_node-------------------")
    print(f"消息列表: {state['messages']}")
    print(f"大模型调用次数: {state['llm_calls']}")
    print("-----------------tool_node-------------------")
    
    for tool_call in state["messages"][-1].tool_calls:
        #1. 获取工具(根据工具映射表获取工具)
        tool = tools_by_name[tool_call["name"]]

        #2. 调用工具
        tool_result = tool.invoke(tool_call["args"])
        
        #3. 构建一个ToolMessage
        tool_message = ToolMessage(
            content=tool_result,
            tool_call_id=tool_call["id"],
        )
        # result.append(tool_message)
        # return {"messages": result}

        #4. 构建状态
        new_state = {
            "messages": [tool_message],
            "llm_calls": state.get("llm_calls", 0),
        }

        return new_state

#6.3 路由决策函数
def should_continue(state: dict) -> bool:
    """
    路由决策函数
    :param state: 状态
    :return: 是否继续
    """
    print(f"消息列表: {state['messages']}")
    print(f"大模型调用次数: {state['llm_calls']}")

    #1. 获取消息
    messages = state["messages"]

    #2. 获取最近一条消息
    last_message = messages[-1]

    #3. 判断最近一条消息是否是ToolMessage
    if last_message.tool_calls:
        return "tool_node"
    
    return END
    
#7. 构建并编译代理（构建状态图）

from langgraph.graph import StateGraph
from langgraph.graph import START, END
#7.1 构建状态图
agent_builder = StateGraph(MessageState)

#7.2 添加节点
agent_builder.add_node("llm_call", llm_call_node)
agent_builder.add_node("tool_node", tool_node)

#7.3 添加边
agent_builder.add_edge(START, "llm_call")           # START->llm_call节点
agent_builder.add_conditional_edges(
    "llm_call",
    should_continue,                                # 条件分支,路由决策函数,决策是否调用工具,如果不用调用工具,直接走END节点
    ["tool_node", END]                              # 如果需要调用工具,走tool_node节点,否则走END节点
)
agent_builder.add_edge("tool_node", "llm_call")     # tool_node->llm_call节点

#7.4 编译状态图
agent = agent_builder.compile()
        
#8. 显示状态图，并保存到一个图片文件中
agent_image = agent.get_graph(xray=True).draw_mermaid_png()
with open("agent_graph.png", "wb") as f:
    f.write(agent_image)

if __name__ == "__main__":
    while True:
        question = input("请输入问题: ")
        if question == "exit":
            break

        #1. 初始化状态
        state = {
            "messages": [],
            "llm_calls": 0,
        }
        state['messages'].append(HumanMessage(content=question))

        #1. 执行代理
        messages = agent.invoke(input=state)
        # print(messages)
        print(messages['messages'][-1].content, end="", flush=True)
        print()


```