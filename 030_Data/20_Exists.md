## 检查文档是否存在

如果你想做的只是检查文档是否存在——你对内容完全不感兴趣——使用`HEAD`方法来代替`GET`。`HEAD`请求不会返回响应体，只有HTTP头：

```Javascript
curl -i -XHEAD http://localhost:9200/website/blog/123
```

Elasticsearch将会返回`200 OK`状态如果你的文档存在：

```Javascript
HTTP/1.1 200 OK
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

如果不存在返回`404 Not Found`：

```Javascript
curl -i -XHEAD http://localhost:9200/website/blog/124
```

```Javascript
HTTP/1.1 404 Not Found
Content-Type: text/plain; charset=UTF-8
Content-Length: 0
```

当然，这只表示你在查询的那一刻文档不存在，但并不表示几毫秒后依旧不存在。另一个进程在这期间可能创建新文档。
