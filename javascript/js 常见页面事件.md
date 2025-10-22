### 1 window.onload

当整个页面（包括所有依赖资源如图片）完全加载完成后触发

```js
window.onload = function() {
  // 页面加载完成后的代码
};
```

### 2 DOMContentLoaded

当DOM树完全构建完成后触发，不需要等待样式表、图片和子框架的加载完成。

```js
document.addEventListener('DOMContentLoaded', function() {
  // DOM加载完成后的代码
});
```

### 3.window.beforeunload

当窗口、文档或其资源即将被卸载时触发。常用于询问用户是否真的要离开

```js

window.onbeforeunload = function() {
  alert('你确定要离开吗？'); // 显示一个对话框询问用户
};
```

### 4 window.onscroll

当文档或元素滚动时触发。常用于创建滚动监听效果，如无限滚动。（时间处罚频繁，可以使用防抖）

```js
window.onscroll = function() {
  // 滚动时的代码
};
```