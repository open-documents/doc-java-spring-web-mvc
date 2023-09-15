在第一次请求模型属性添加进session后，会一直保存在里面，知道其他控制器方法通过SessionStatus#setComplete()方法清空。

