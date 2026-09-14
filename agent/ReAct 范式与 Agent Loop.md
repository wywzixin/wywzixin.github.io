**Agent Loop 就是让模型学会「自己干活」的那个机制**

****

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169423770-02de843f-917a-47a2-b879-5c4df0387dd5.png)



## ReAct思想


<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169524458-bd486ac9-53af-43c6-a4b8-eec2ce7eed4e.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169542317-b9327435-6c1f-4f1b-b70a-b7925aa1a336.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169557521-14fecb46-7b5c-4022-8dad-558507d46219.png)



## Agent Loop 的核心：就是一个 while 循环
```python
function agentLoop(userMessage):
    messages = [...历史消息, userMessage]
    while true:
        response = callLLM(systemPrompt, messages, tools)
        if response 没有 tool_use:
            return response
        messages.append({role: "assistant", content: response.content})
        results = []
        for each toolUse in response.toolUses:
            result = executeTool(toolUse.name, toolUse.input)
            results.append(tool_result(toolUse.id, result))
        messages.append({role: "user", content: results})
```



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169641277-7538668d-de92-4de6-a917-d44de75fc414.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169683487-90e9b032-287e-41fe-9a33-f993f45a0708.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169717713-7d1c862c-ab0c-4439-8fa2-f72c1123a1e0.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169757224-b97d95e1-048c-4be8-9e08-0d46c1b537ff.png)





## AgentEvent 流


<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169857472-5d9b463c-2f6b-4bb3-8b1d-398e6f2f1d52.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169871910-73c933af-d525-4853-83c2-e2f5875acd89.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784169956899-d33b7616-4422-4d27-8250-14a0ba1fb7f8.png)







<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784170028313-a3bb6753-6596-4577-b9fb-1457ae81bc96.png)





## plan模式


```plain
Plan mode is active. 你不能执行任何修改操作，不能编辑文件、不能提交代码、不能修改配置。
唯一可以写入的文件是下面指定的 plan file。

你的工作流程：
1. 用 ReadFile、Grep、Glob、Bash（只读命令）探索代码
2. 分析用户需求，设计实现方案
3. 把计划写入 plan file
4. 等待用户确认后再执行
```



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784170164971-f558a367-cb0a-4ab5-a16b-37e1683334d7.png)





## 数据流
llm_stream

    ↓

底层模型事件 StreamEvent

    ↓

collector.consume()

    ↓

上层 AgentEvent

    ↓

当前这个 async for

















