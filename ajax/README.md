实现跨域的几种方法：
- JSONP
  1. 前端定义好函数声明`fn`
  2. 后端返回**函数调用`fn(res)`形式的字符串**，res相当于平时ajax请求接口时返回的数据
  3. 前端通过script标签请求第二部里后端返回的字符串，由于是script标签，就相当于是执行了一遍函数调用`fn(res)`。
  4. 缺点：只有get方法能使用JSONP

  ```
  // 前端
  <script src="xxx">
  <script>
  function fn(res){
    console.log(res)
  }
  </script>

  // 后端
  res.write(`fn({code: 0, data: {}})`)
  ```

- CORS
- postMessage
- nginx
- nodeJs中间件
- webpack-dev-server代理接口

参考：
[前端常见跨域解决方案（全）](https://segmentfault.com/a/1190000011145364#articleHeader1)