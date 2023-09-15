	参考文档:P951

其中的许多RequestPredicate是合并的RequestPredicate，如
- RequestPredicates.GET(String) -- RequestPredicates.method() 和 RequestPredicates.path(String)

# 静态方法

| 静态方法 | 参数         | 返回值                  | 描述                                                 |
| -------- | ------------ | ----------------------- | ---------------------------------------------------- |
| accept() | MediaType... | AcceptPredicate实例     | 返回实例用以测试请求头中的Accept是否满足MediaType... |
| method() | HttpMethod   | HttpMethodPredicate实例 | 返回实例用以测试请求的方法是否为参数中的方法         |
|          |              |                         |                                                      |


