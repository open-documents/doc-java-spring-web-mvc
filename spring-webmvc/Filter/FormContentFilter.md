	参考文档:P884

浏览器能够通过HTTP GET和HTTP POST的方式提交表单数据，非浏览器客户端另外能够使用HTTP PUT，HTTP PATCH，HTTP DELETE提交表单数据。但是`ServletRequest.getParameter*()`只能获取通过HTTP POST方式提交的请求的表单域。

FormContentFilter能够拦截通过HTTP PUT，HTTP PATCH，HTTP DELETE方式提交的并且content type内容为application/x-www-form-urlencoded的表单，从请求体中获得数据，并对ServletRequest进行包装，使得`ServletRequest.getParameter*()`方法簇能够获取表单数据。

