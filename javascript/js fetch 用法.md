‌Fetch API 是 JavaScript 中用于替代XMLHttpRequest 的现代网络请求接口‌，基于 Promise 实现异步通信，支持 GET/POST 等多种 HTTP 方法，并兼容 Service Worker 等场景。‌

### 核心功能 

1.‌替代传统方案‌：作为 XMLHttpRequest 的现代化改进方案，提供更简洁的语法和链式调用。‌

2.‌HTTP接口抽象‌：通过 Request/Response 对象封装HTTP协议细节，支持 Headers、Body 等模块化操作。‌

‌3.应用场景扩展‌：可在 Service Worker 中拦截请求，与 Cache API 结合实现离线资源管理。‌

### 使用与注意事项

基础语法‌：```fetch(url, { method:'GET', headers:{}, body:JSON.stringify(data) })，```返回 ```Promise``` 对象。‌

‌响应处理‌：需两次 ```.then()``` 调用（首次解析HTTP头，二次获取响应体数据），可通过 ```response.json()``` 转换 JSON 格式。‌

‌兼容性问题‌：IE浏览器不支持，需使用 ```polyfill``` 或 ```axios``` 等替代方案。‌‌

‌错误处理‌：需手动检查 ```response.ok``` 状态，网络错误会触发 ```catch```。‌
