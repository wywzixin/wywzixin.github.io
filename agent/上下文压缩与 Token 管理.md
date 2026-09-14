

---





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784253879895-68c2d01a-cec6-485b-bb6b-0266d2a9a50d.png)





## 大结果存磁盘


<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784253937648-594c9676-a0ec-4794-84c9-6efe4745876d.png)



:::tips
<persisted-output>

输出太大（80KB），完整内容已保存到：

.mewcode/session/tool-results/toolu_abc123.txt



预览（前 2KB）：

=== 测试运行结果 ===

PASS: TestUserCreate (0.02s)

PASS: TestUserUpdate (0.01s)

FAIL: TestUserDelete (0.03s)

    expected: nil, got: permission denied

...

</persisted-output>

:::

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784254117772-53ec41a9-216c-4fc1-b6a0-689da3e81822.png)



MewCode 采用的解决方案叫「决策冻结」，这也是 Claude Code 使用的策略。系统维护一个 `ContentReplacementState` ，记录每个 `tool_use_id` 的替换决策。规则很简单：

**一旦某个工具结果在某一轮被决定「不替换」，这个决定在整个会话内永远不变。 **即使后来按聚合限制它应该被替换，也不替换，因为替换会破坏前缀。反过来，如果决定了「替换」，那每一轮都用完全一样的预览字符串去替换，确保前缀逐字节不变。

 一个工具结果第一次出现时，系统只判断一次：保留原文，还是替换成预览。决定之后，永远不反悔。  

每一轮面对工具结果时，系统把它们分成三类：



:::tips
已替换过的    → 用缓存的预览字符串原样重放，不重新生成

已决定不替换的 → 永远保留原文，不再评估

新产生的      → 正常评估是否需要替换，做出决策后冻结

:::





**上下文管理不仅要考虑「保留什么信息」，还要考虑「改动对话前缀的代价**

****

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784254887865-d82522df-8506-4b32-a17b-25d74562fae3.png)



## 全量摘要


**调用 LLM 自己来生成一份对话摘要**

****

:::tips
上下文窗口               200,000

 - 预留给摘要输出         - 20,000    摘要本身也要占空间

= 有效窗口              180,000

 - 安全余量              - 13,000    防止 Token 估算误差导致临界抖动

= 自动压缩阈值          167,000     超过这个数就触发全量摘要

:::





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255022184-bddd5177-2f52-4114-96b3-f169dc0eff69.png)

### 摘要 Prompt 的设计
:::tips
1. 主要请求和意图        用户到底想做什么

2. 关键技术概念          讨论过的重要技术点

3. 文件和代码段          涉及哪些文件，关键代码片段要保留

4. 错误和修复            遇到了什么错，怎么解决的

5. 问题解决过程          解决问题的思路和方法

6. 所有用户消息          用户说过的所有非工具结果的话（原文保留！）

7. 待办任务              还没完成的事

8. 当前工作              最近在做什么（要最详细）

9. 可能的下一步          接下来打算做什么

:::

**用户说过的话尽量原文保留 **。为什么不能摘要改写？因为用户的原始表述包含意图、偏好、语气。如果用户说「不要用 interface{}，用 any」，你把它摘要成「用户偏好现代语法」，模型下次可能给出 `type any = interface{}` 这种不是用户想要的东西

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255158629-6f010fa0-4b6d-4b88-aa97-2d5ec7ac2bd4.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255198780-1442bbad-be08-43f2-b1da-90813631404f.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255263372-25342ace-a9ec-4b7a-a45d-7cf4a5e64c8a.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255291937-fb69cb72-5b0c-43cf-9c28-6f94be64a188.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255319124-55e047f1-66d2-4475-b6a0-9e1664ad0ed6.png)

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255375808-e2075704-a3be-4ea1-a781-c78691cce39c.png)







<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255392864-a68d3df3-7f01-4bd2-af86-b3fa66ad38a8.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255466835-9ce25db1-8cfe-4619-abfe-1ae51a4ca9d2.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784255476700-5562cf0d-1bbb-4ca1-a5a6-7757a46bc645.png)





## 熔断器


**熔断器（Circuit Breaker）** 是一种常见的容错设计模式，这里用在自动压缩上。

### 它在这个项目里的具体实现
Python

```plain
@dataclass
class CompactCircuitBreaker:
    max_failures: int = 3
    consecutive_failures: int = field(default=0, init=False)

    def record_failure(self) -> None:
        self.consecutive_failures += 1

    def record_success(self) -> None:
        self.consecutive_failures = 0

    def is_open(self) -> bool:
        return self.consecutive_failures >= self.max_failures
```

逻辑非常简单：

+ 压缩**失败一次** → consecutive_failures + 1
+ 压缩**成功一次** → 计数清零
+ 连续失败达到 **3 次** → 熔断器打开（is_open() == True）

### 它解决什么问题？
自动压缩本身是一次**额外的大模型调用**（要生成摘要），会消耗 token 和时间。

如果因为某些原因（提示词太长、模型拒绝、网络问题、摘要质量差等）导致压缩连续失败，系统如果还一直重试，就会：

1. 浪费大量 token
2. 拖慢响应速度
3. 在已经很紧张的上下文里反复失败，最终还是撑爆

所以加了熔断器：**连续失败 3 次后，暂时停止自动压缩**，返回提示让用户手动处理（用 /compact）。

### 使用位置
Python

```plain
# 处于软硬阈值之间时
if breaker is not None and breaker.is_open():
    return "自动压缩已熔断（连续失败 3 次），请手动处理或使用 /compact"
```

注意：**只有在软阈值和硬阈值之间才会检查熔断器**。

一旦超过硬阈值，会**强制压缩，无视熔断器**（因为再不压就真的要炸了）。

### 简单类比
| 状态 | 熔断器行为 | 类比 |
| --- | --- | --- |
| 正常 | 允许自动压缩 | 电路正常通电 |
| 连续失败 3 次 | 打开熔断，停止自动压缩 | 跳闸，保护线路 |
| 手动压缩成功 / 强制压缩 | 清零计数 | 重新合闸 |


这就是为什么代码里叫 CompactCircuitBreaker——专门保护「自动压缩」这条容易失败的路径。

