（20个常用 Object 方法，含简述与示例）：

- Object.keys(obj): 返回可枚举自有属性键数组
```js
Object.keys({a:1,b:2}) // ["a","b"]
```
- Object.values(obj): 返回可枚举自有属性值数组
```js
Object.values({a:1,b:2}) // [1,2]
```
- Object.entries(obj): 返回 [key,value] 数组
```js
Object.entries({a:1,b:2}) // [["a",1],["b",2]]
```
- Object.fromEntries(iterable): 由可迭代的键值对构造对象
```js
Object.fromEntries([["a",1],["b",2]]) // {a:1,b:2}
```
- Object.assign(target, ...sources): 浅拷贝/合并到 target
```js
Object.assign({}, {a:1}, {b:2}) // {a:1,b:2}
```
- Object.create(proto, [propertiesObject]): 以原型 proto 创建对象
```js
const o = Object.create(null); Object.getPrototypeOf(o) // null
```
- Object.defineProperty(obj, key, descriptor): 定义/修改单个属性
```js
const o={}; Object.defineProperty(o,"x",{value:1,writable:false})
```
- Object.defineProperties(obj, descriptors): 批量定义属性
```js
Object.defineProperties({}, {a:{value:1}, b:{value:2}})
```
- Object.getOwnPropertyDescriptor(obj, key): 获取属性描述符
```js
Object.getOwnPropertyDescriptor({a:1},"a")
// { value:1, writable:true, enumerable:true, configurable:true }
```
- Object.getOwnPropertyDescriptors(obj): 获取所有自有属性描述符
```js
Object.getOwnPropertyDescriptors({a:1,b:2})
```
- Object.getOwnPropertyNames(obj): 自有字符串键（含不可枚举）
```js
Object.getOwnPropertyNames(Object.freeze({a:1})) // ["a"]
```
- Object.getOwnPropertySymbols(obj): 自有 symbol 键
```js
const s=Symbol("k"); const o={[s]:1}; Object.getOwnPropertySymbols(o) // [s]
```
- Object.hasOwn(obj, key): 是否为“自有”属性（不查原型链）
```js
Object.hasOwn({a:1},"a") // true
```
- Object.getPrototypeOf(obj): 获取原型
```js
Object.getPrototypeOf([]) === Array.prototype // true
```
- Object.is(a, b): 更精确的 SameValue 比较
```js
Object.is(NaN, NaN) // true; Object.is(0, -0) // false
```
- Object.freeze(obj): 冻结对象（不可扩展、属性不可重配置/写）
```js
const o=Object.freeze({a:1})
```
- Object.seal(obj): 密封对象（不可扩展，现有属性不可删）
```js
const o=Object.seal({a:1})
```
- Object.preventExtensions(obj): 禁止扩展（不可新增属性）
```js
const o=Object.preventExtensions({a:1})
```
- Object.isExtensible(obj): 是否可扩展
```js
Object.isExtensible({}) // true
```
- Object.getOwnPropertySymbols + keys/values/entries 组合：遍历完整键集合时常一起使用
```js
const ownKeys = [
  ...Object.getOwnPropertyNames(obj),
  ...Object.getOwnPropertySymbols(obj),
];
``` 

请告知当前文件路径，或直接让我把以上内容写入该“打开的文件”。