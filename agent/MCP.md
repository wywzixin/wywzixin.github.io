```python
  tools = self.registry.get_all_schemas(self.protocol)
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784870333532-4e0aa914-33d0-4bef-bce0-59e8433a3af7.png)







---

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216595366-31d8d324-c926-485c-a372-bc37f7bafb88.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216621817-0750506c-8cab-4202-8e7c-8e6fd205ed3b.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216790720-fead2842-e743-49bb-9e14-08e2d8793077.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216824056-6fdeff62-2083-4d88-85b3-8ae00e6681b7.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216875398-24643272-4bc0-41d1-b70c-6f91b085535a.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216885139-0f76d758-59e8-4e46-9e14-5f8aaaa22b6b.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216904044-48febe9a-d495-422f-905b-1634a196c242.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216923664-c07c40e7-98d3-4fd9-af0c-fa7a141fac82.png)







<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784216965715-1f2d999b-bea2-43d4-915f-8bfedaafbf1d.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217009660-c17e4bb6-3b68-4383-b3d0-5e0fb0dc78f7.png)



```python
mcp_servers:
# stdio：有 command 字段 → 启动子进程，走管道
github:
command: "npx"
args: ["-y", "@modelcontextprotocol/server-github"]
env:
GITHUB_TOKEN: "${GITHUB_TOKEN}"

# Streamable HTTP：有 url 字段 → 发 HTTP 请求
remote-tool:
url: "https://api.example.com/mcp"
headers:
Authorization: "Bearer ${API_TOKEN}"
```

<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217070354-a98fe14f-1d44-4d42-b281-36e1b3ef21b8.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217168185-634de730-d0c9-4641-9f16-7dd3bef4264d.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217206609-cc713356-7e6c-40c7-acf4-631aa925ac2b.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217249151-6e6ffdf4-2827-45cd-b3d8-69319a3ee695.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217266560-069da23a-8219-4d79-abd3-6bf15b503011.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217284906-7b9badf2-98fc-4b25-be25-7e5ebe753996.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217317776-f863e285-d26f-4663-8c89-21bc80f9ee39.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217405630-c3d5a60d-20d7-4203-bb3b-d68116257721.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217447526-bf8060a8-1f08-4ce8-9c0e-836b5c114dc2.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217536245-1ee614f2-12b3-4b9c-b682-3af98c800ad5.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217658291-6a3a8532-104b-467f-af97-332f1d247696.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217740349-90f01799-399d-4059-afd8-d4f2654f615e.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217754387-28539b35-685b-4dd7-a05b-bc68af5bfa3e.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217812315-da9b0d57-524d-4dfc-b0b2-b34b301bf826.png)





<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217844600-82a8fcf7-e123-4fed-94b6-bc02c28ab54c.png)



<!-- 这是一张图片，ocr 内容为： -->
![](https://cdn.nlark.com/yuque/0/2026/png/276041/1784217973964-48042171-aaef-4cbc-bcb8-29ff740422e0.png)









