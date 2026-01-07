---
date: 2026-01-06
---
# 概述
  
hasattr 的身份是 **Python 的内置函数 (Built-in Function)**。

# 作用
判断一个对象是否有某种属性
- 举例(判断request是否含有messages这个属性)
```
if hasattr(request, 'messages') and request.messages:
        # 从后往前遍历，只取第一条（最新）的HumanMessage
        for msg in reversed(request.messages):
            if isinstance(msg, HumanMessage) and msg.content.strip():
                human_prompt = msg.content
                break
```

# 其他类似的函数及用法

- hasattr(obj,'name'):**检查**是否有这个属性
- getattr(obj,'name'):**获取**这个属性的值
- setattr(obj,'name',value):**设置**该属性的值
- delattr(obj,'name'):**删除**属性


# 补充知识

## 如何返回一个函数对象
```
# 定义两个普通的小函数
def say_hello(name):
    return f"Hello, {name}!"

def say_bye(name):
    return f"Goodbye, {name}!"

# 定义一个主函数，它返回的是函数对象，而不是字符串
def get_greeting_func(style):
    if style == "start":
        return say_hello  # 注意：这里没有括号 ()
    else:
        return say_bye    # 注意：这里没有括号 ()

# --- 使用 ---

# 1. 获取函数
my_func = get_greeting_func("start") 

# 打印看看 my_func 是什么
print(my_func) 
# 输出: <function say_hello at 0x...> (说明它是个函数对象)

# 2. 调用这个拿到手的函数
result = my_func("Alice")
print(result) 
# 输出: Hello, Alice!
```