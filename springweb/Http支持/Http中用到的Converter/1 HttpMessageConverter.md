`HttpMessageConverter<T>` 是 Spring3.0 新添加的一个接口，负责将请求信息转换为一个对象（类型为 T），**将对象（类型T）输出为响应信息**；

### 　　2、HttpMessageConverter<T> 接口定义的方法

　　① Boolean canRead(Class<?> clazz, MediaType mediaType)：指定转换器可以读取的对象类型，即转换器是否可将请求信息转换为 clazz 类型的对象，同时指定支持 MIME 类型（text/html; application/json 等）

        ② Boolean canWriter(Class<?> clazz, MediaType mediaType)：指定转换器是否可将 clazz 类型的对象写到响应流中，响应流支持的媒体类型在 MediaType 中定义；

        ③ List<MediaType> getSupportMediaTypes()：该转化器支持的媒体类型；

        ④ T read(Class<? extends T> clazz, **HttpInputMessage** inputMessage )：将请求信息转换为 T 类型的对象；

        ⑤ void write(T t, MediaType contentType, **HttpOutputMessage** outputMessage)：将 T 类型的对象写到响应流中，同时指定相应的媒体类型为 contentType。