

## Claude请求和回复
```python
// ==================== 1. 发送给 Claude 的请求 ====================
{
  "model": "claude-sonnet-5",
  "max_tokens": 16000,
  "system": [
    {
      "type": "text",
      "text": "你是一个专业的编程助手",
      "cache_control": {"type": "ephemeral"}
    }
  ],
  "thinking": {
    "type": "adaptive",
    "display": "summarized"
  },
  "tools": [
    {
      "name": "get_weather",
      "description": "获取指定城市的天气",
      "input_schema": {
        "type": "object",
        "properties": {
          "city": {"type": "string"}
        },
        "required": ["city"]
      }
    }
  ],
  "messages": [
    {
      "role": "user",
      "content": "东京现在天气怎么样？"
    },
    {
      "role": "assistant",
      "content": [
        {
          "type": "thinking",
          "thinking": "用户想知道东京天气，我需要调用天气工具",
          "signature": "EqoBCkgIARABGAIiQ..."
        },
        {
          "type": "text",
          "text": "好的，我来帮你查询东京的天气。"
        },
        {
          "type": "tool_use",
          "id": "toolu_01A09q90qw90lq917835lq9",
          "name": "get_weather",
          "input": {
            "city": "Tokyo"
          }
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "tool_result",
          "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
          "content": "东京：晴天，28°C，湿度 65%",
          "is_error": false
        }
      ]
    }
  ]
}
```

```python
// ==================== 2. Claude 返回的完整回复 ====================
{
  "id": "msg_01XFDUDYJgAACzvnptvVoYEL",
  "type": "message",
  "role": "assistant",
  "model": "claude-sonnet-5",
  "content": [
    {
      "type": "thinking",
      "thinking": "工具已经返回了天气数据，我可以直接告诉用户结果了。",
      "signature": "EqoBCkgIARABGAIiQ..."
    },
    {
      "type": "text",
      "text": "东京现在是晴天，气温 28°C，湿度 65%。天气不错，适合出门。"
    }
  ],
  "stop_reason": "end_turn",
  "stop_sequence": null,
  "usage": {
    "input_tokens": 1245,
    "output_tokens": 86,
    "cache_creation_input_tokens": 0,
    "cache_read_input_tokens": 3200
  }
}
```

```python
{
  "role": "user",
  "content": [
    {
      "type": "tool_result",
      "tool_use_id": "toolu_01A09q90qw90lq917835lq9",
      "content": "工具执行结果的文本"
    }
  ]
}

```

## 传的参数


```javascript
{
  "id": "msg_013Zva2CMHLNnXjNJJKqJ2EF",
  "type": "message",
  "role": "assistant",
  "content": [
    {
      "type": "text",
      "text": "Hello! How can I help you today?"
    }
  ],
  "model": "claude-sonnet-4-6",
  "stop_reason": "end_turn",
  "usage": {
    "input_tokens": 10,
    "output_tokens": 12
  }
}
```



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784040108765-f291ed6e-8cb1-4c56-9c09-e428292d5f0f.png)





```javascript
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 4096,

  "system": "你是 MewCode，一个终端环境中的 AI 编程助手。\n\n# Environment\n当前工作目录: /home/dev/myproject\n操作系统: Linux\n当前时间: 2026-05-27",

  "messages": [
    {"role": "user", "content": "帮我读一下 app.py 的内容"},
    {"role": "assistant", "content": "好的，我来读取 app.py 的内容。\ndef main():\n    print(\"hello\")\n\nif __name__ == \"__main__\":\n    main()"},
    {"role": "user", "content": "这个文件里有什么函数？"}
  ],

  "tools": [
    {
      "name": "read_file",
      "description": "读取指定路径的文件内容",
      "input_schema": {
        "type": "object",
        "properties": {
          "path": {"type": "string", "description": "文件路径"}
        },
        "required": ["path"]
      }
    }
  ]
}
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784040303000-388947a0-e2ca-467b-b12c-576f85b686e6.png)

LLM 返回一个工具调用请求，这算 assistant 的消息。你执行完工具拿到了结果，需要把结果发回去—— 这个结果要作为 user 消息发送 ，因为从 API 的视角看，所有你发给模型的东西都归 user，模型返回的都归 assistant。如果你搞错了角色，把工具结果也当成 assistant 发出去，就会出现两条 assistant 连在一起，API 会直接拒绝：



```javascript
{
  "type": "error",
  "error": {
    "type": "invalid_request_error",
    "message": "messages: roles must alternate between \"user\" and \"assistant\", but found multiple \"assistant\" roles in a row"
  }
}
```



---

## 流式响应
流式响应基于 SSE（Server-Sent Events）协议



:::tips
message_start          整个响应开始，带着 input_tokens 信息

  └─ content_block_start   一个内容块开始（文本或工具调用）

       └─ content_block_delta  内容增量，文字一个词一个词地到达

  └─ content_block_stop    一个内容块结束

message_delta            消息级别的增量（output_tokens、停止原因）

message_stop             整个响应结束

:::

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784083351399-222b2c6a-d7a6-477f-8b86-0de292b70c65.png)



```python
async for event in stream:
    if event.type == "content_block_start":          # 当一个新的内容块开始时
        block = event.content_block                 # 把这个内容块拿出来
        if block.type == "thinking":                # 判断这个块是不是“思考”
            in_thinking = True
            thinking_accum = ""
            thinking_signature = ""
        elif block.type == "tool_use":              # 判断是不是“工具调用”
            ...
```



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784096153401-40ae8239-3777-4bce-848f-2cda022789b4.png)

```python
elif event.type == "content_block_delta":
                        delta = event.delta
                        if delta.type == "text_delta":
                            yield TextDelta(text=delta.text)
                        elif delta.type == "thinking_delta":
                            thinking_accum += delta.thinking
                            yield ThinkingDelta(text=delta.thinking)
                        elif delta.type == "signature_delta":
                            thinking_signature = delta.signature
                        elif delta.type == "input_json_delta":
                            json_accum += delta.partial_json
                            yield ToolCallDelta(text=delta.partial_json)
```

**这 4 种 ****delta.type**** 分别是什么意思？**

当你收到 event.type == "content_block_delta" 时，说明 Claude 正在**增量更新**某个内容块。 delta 就是这次更新的具体内容，delta.type 决定了它是哪种更新。

下面按你贴的代码顺序解释：

---

### 1. text_delta —— 普通文字增量（最常见）
Python

```python
if delta.type == "text_delta":
    yield TextDelta(text=delta.text)
```

**含义**：最终回答的文字片段。

+ 字段：delta.text
+ 例子："Hello", " world", "!" 等
+ 这是用户最终看到的回复内容

**用途**：实时把文字显示到网页上（打字机效果）。

---

### 2. thinking_delta —— 思考过程增量
Python

```python
elif delta.type == "thinking_delta":
thinking_accum += delta.thinking
yield ThinkingDelta(text=delta.thinking)
```

**含义**：Claude 的**内部思考过程**的文字片段（开启 Thinking 后才会有）。

+ 字段：delta.thinking
+ 例子："我需要用欧几里得算法来计算最大公约数..."
+ 通常在最终回答之前出现

**用途**：实时展示 Claude 的思考过程（可以做成可折叠的思考区域）。

---

### 3. signature_delta —— 思考签名（加密验证）
Python

```python
elif delta.type == "signature_delta":
thinking_signature = delta.signature
```

**含义**：Thinking 内容的**加密签名**（用于验证思考内容没有被篡改）。

+ 字段：delta.signature（一长串加密字符串）
+ 出现时机：通常在 thinking 块快结束时（content_block_stop 之前）出现一次
+ 一般**不需要展示给用户**，只是存起来用于后续验证或传递

**用途**：安全/完整性验证（高级场景才会用到）。

---

### 4. input_json_delta —— 工具调用参数增量（Tool Use）
Python

```python
elif delta.type == "input_json_delta":
json_accum += delta.partial_json
yield ToolCallDelta(text=delta.partial_json)
```

**含义**：Claude 调用工具时，**工具参数（JSON）的碎片**。

+ 字段：delta.partial_json
+ 例子：text

```plain
"{\"location\": \"San "
"Francisco\", \"unit\": \"celsius\"}"
```

+ 因为 JSON 是一点点生成的，所以要自己累加（json_accum += ...）

**用途**：

+ 实时显示工具调用的参数
+ 等整个 JSON 完整后，用 json.loads(json_accum) 解析成字典，然后真正执行工具

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784096420789-d80a901a-e2fc-4557-92b7-579eaf7ba6d0.png)



`yield` 关键字是 async generator 的核心。每次 yield 都暂停当前函数，把事件交给调用方处理。调用方处理完了，执行权返回这里继续读下一个 SSE 事件。这种协作式调度非常轻量，没有缓冲区管理的心智负担，天然就是背压友好的。

---







---

```javascript
{
  "model": "claude-sonnet-4-6",
  "max_tokens": 4096,

  "system": "你是 MewCode，一个终端环境中的 AI 编程助手。\n\n# Environment\n当前工作目录: /home/dev/myproject\n操作系统: Linux\n当前时间: 2026-05-27",

  "messages": [
    {"role": "user", "content": "帮我读一下 app.py 的内容"},
    {"role": "assistant", "content": "好的，我来读取 app.py 的内容。\ndef main():\n    print(\"hello\")\n\nif __name__ == \"__main__\":\n    main()"},
    {"role": "user", "content": "这个文件里有什么函数？"}
  ],

  "tools": [
    {
      "name": "read_file",
      "description": "读取指定路径的文件内容",
      "input_schema": {
        "type": "object",
        "properties": {
          "path": {"type": "string", "description": "文件路径"}
        },
        "required": ["path"]
      }
    }
  ]
}
```





---





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784040650348-b6a4c576-518b-40ba-8fd3-d1385a926c66.png)

**API 层 **还是那个 `role` + `content` ，专门用来跟 LLM 通信，保持简单干净。

**内部层 **则丰富得多。角色从两种扩展到四种： `user` 、 `assistant` 、 `system` 、 `tool` 。<font style="background-color:rgba(255,246,122,0.8);">每条</font><font style="background-color:rgba(255,246,122,0.8);">消息带一个唯一 ID</font>，方便定位和更新。还有时间戳、Token 用量、响应耗时这些元数据。

最关键的是多了一个状态字段。一条 assistant 消息刚创建时是 `streaming` ，流式接收完毕变成 `complete` ，出错变成 `error` 。



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784040777817-ba187b79-e093-4614-bd3c-4655916a453f.png)、

MewCode 采用 TUI（Terminal User Interface）方案



## Thinking 参数


**Claude Thinking（扩展思考）传参完整指南（2026 最新）**

Thinking 是 Anthropic 的**内部推理**功能，让 Claude 先思考再回答，效果明显更好（尤其是复杂问题）。



```python
response = client.messages.create(
    model="claude-sonnet-5",          # 或 opus-4-8 / fable-5 等
    max_tokens=16000,
    thinking={
        "type": "adaptive",           # 推荐用这个
        "display": "summarized"       # 是否返回思考内容
    },
    messages=[{"role": "user", "content": "你的问题"}]
)
```



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784084712738-db6e7bb0-a330-4e62-939a-3dcadac58984.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784084878362-ecd28f54-934e-4757-a16b-e2cf6684d246.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784084918262-52ff2b11-4fd1-4e0c-951a-0c62b4812e16.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784084957199-7f16959b-7937-41f1-bdca-34d588a75b9a.png)



## 异常处理
```python
except _anthropic.AuthenticationError as e:
    raise AuthenticationError(f"Invalid API key: {e}") from e
except _anthropic.RateLimitError as e:
    retry = e.response.headers.get("retry-after") if e.response else None
    raise RateLimitError(
        f"Rate limited. {f'Retry after {retry}s.' if retry else 'Please wait.'}",
        retry_after=float(retry) if retry else None,
    ) from e
except _anthropic.APIConnectionError as e:
    raise NetworkError(f"Network error: {e}") from e
except _anthropic.APIStatusError as e:
    raise LLMError(f"API error ({e.status_code}): {e.message}") from e
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784105633529-2b6a72a0-ff39-4f6a-8841-07fc520c10d5.png)

