讲 reactive 和 ref 前，先了解 vue3的数据响应的核心：Proxy，通过创建一个对象的代理，从而实现对此对象操作的拦截和自定义。因为 Proxy 本身返回一个新对象。
```javascript
// 定义一个对象
let obj = {
  name: '守候'
};

// 将对象传入 Proxy 中，代理重写 get 和 set 方法
const proxyObj = new Proxy(obj, {
  get: (target,key) => {
    console.log(`获取属性 ${key} 数据`);
    return target[key]
  },

  set: (target,key,value) => {
    console.log(`把对象的 ${key} 属性值设置为 ${value}`);
    target[key]=value
  }
});

proxyObj.name = '修改后的数据';
//把对象的 name 属性值设置为 修改后的数据

proxyObj.name
//获取属性 name 数据
//'修改后的数据'
```
因为语法上的限制，仅对对象类型有效（对象、数组和 Map、Set 这样的集合类型），对 string、number 和 boolean 这样的原始类型无效，没法操作原始值本身。

//把 obj 改成字符串再执行下面的代码
```javascript
let obj = '守候';

//Uncaught TypeError: Cannot create proxy with a non-object as target or handler
```
控制台输出了如下报错信息：无法使用非对象作为目标或处理程序创建代理。这意味着 Proxy 是无法对原始数据类型进行代理的。

而 vue3中的 reactive 正是基于 Proxy 封装而成的，所以 reactive 不能监听原始数据类型也就不难理解了。

那为什么 ref 可以监听原始数据类型？因为 ref 是基于 reactive 封装的。

可以简单的理解为：ref(‘守候’) 就是 reactive({ value: ‘守候’ })，这就解释了为什么 ref 能监听原始数据类型，以及为什么 ref 的数据在获取值的时候要加 .value, 至于在 html 模板那里为什么不用加 .value，其实是 vue3 做了处理。

说到这，如果想要 reactive 支持原始类型，再包一层就好。reactive({ value: ‘守候’ })。

实际上的 ref 的实现比 reactive({ value: X }) 复杂，vue3内部有一个 RefImpl 类，还加入了 __v_isRef 等标记位来识别这个对象是否是 ref 对象，也会用 isObject 接收到的 value 是否为 object，判断。

既然都用到了 reactive，vue 中为什么要有 ref，只要一个 reactive 不就够了嘛？
如果一个组件中的数据量很少，或者说数据都是原始类型的，用 ref 这种原生的 getter，setter，性能上比 reactive 里面的递归代理更高。

既然 ref 支持的类型更多，而且也是基于 reactive ，一般来说尽可能用 ref 就好。但是如果数据比较复杂（data.value.keys.key），或者数据又有 value 属性（data.value.key.value），看着就会比较乱，所以该用 reactive 就还是用 reactive 吧。

在 Vue 3中，reactive 和 ref 都是用于创建响应式数据的 API，但它们在性能上存在一些差异。这些差异主要源于它们的实现方式、使用场景以及性能开销。

综上所述

‌reactive 的优势‌：能够自动处理嵌套属性的响应式，适合管理复杂的嵌套对象和数组。在需要管理多个属性和嵌套结构的状态时，reactive 是更好的选择。

‌ref 的优势‌：性能开销较小，适合处理单一的基本类型数据。在需要频繁更新简单值或进行表单输入双向绑定时，ref 是更好的选择。

参考链接

https://juejin.cn/post/7265999062243213372