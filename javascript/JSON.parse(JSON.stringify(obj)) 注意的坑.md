
JSON.stringify() 几大特性

#### 1.对于 undefined、函数以及 symbol 三种类型的值分别作为对象属性的值、数组元素、单独的值时 JSON.stringify() 将返回不同的结果。


undefined、任意的函数以及 symbol 作为对象属性值时 JSON.stringify() 跳过（忽略）对它们进行序列化

```js
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  }
};
JSON.stringify(data); // 输出：？

// "{"a":"aaa"}"
```

undefined、任意的函数以及 symbol 作为数组元素值时，JSON.stringify() 将会将它们序列化为 null

```js
JSON.stringify(["aaa", undefined, function aa() {
    return true
  }, Symbol('dd')])  // 输出：？

// "["aaa",null,null,null]"
```

ndefined、任意的函数以及 symbol 被 JSON.stringify() 作为单独的值进行序列化时都会返回 undefined

```js
JSON.stringify(function a (){console.log('a')})
// undefined
JSON.stringify(undefined)
// undefined
JSON.stringify(Symbol('dd'))
// undefined
```

#### 2.非数组对象的属性不能保证以特定的顺序出现在序列化后的字符串中。

JSON.stringify() 序列化时会忽略一些特殊的值，所以不能保证序列化后的字符串还是以特定的顺序出现（数组除外）

```js
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  d: "ddd",
  fn: function() {
    return true;
  },
};
JSON.stringify(data); // 输出：？
// "{"a":"aaa","d":"ddd"}"

JSON.stringify(["aaa", undefined, function aa() {
    return true
  }, Symbol('dd'),"eee"])  // 输出：？

// "["aaa",null,null,null,"eee"]"
```

#### 3.转换值如果有 toJSON() 函数，该函数返回什么值，序列化结果就是什么值，并且忽略其他属性的值。

```js
JSON.stringify({
    say: "hello JSON.stringify",
    toJSON: function() {
      return "today i learn";
    }
  })
// "today i learn"
```

#### 4.JSON.stringify() 将会正常序列化 Date 的值

实际上 Date 对象自己部署了 toJSON() 方法（同Date.toISOString()），因此 Date 对象会被当做字符串处理

```js
JSON.stringify({ now: new Date() });
// "{"now":"2019-12-08T07:42:11.973Z"}"
```

#### 5.NaN 和 Infinity 格式的数值及 null 都会被当做 null

```js
JSON.stringify(NaN)
// "null"
JSON.stringify(null)
// "null"
JSON.stringify(Infinity)
// "null"
```

#### 6.布尔值、数字、字符串的包装对象在序列化过程中会自动转换成对应的原始值。

```js
JSON.stringify([new Number(1), new String("false"), new Boolean(false)]);
// "[1,"false",false]"
```

#### 7.仅会序列化可枚举的属性

其他类型的对象，包括 Map/Set/WeakMap/WeakSet，仅会序列化可枚举的属性。

```js
// 不可枚举的属性默认会被忽略：
JSON.stringify( 
    Object.create(
        null, 
        { 
            x: { value: 'json', enumerable: false }, 
            y: { value: 'stringify', enumerable: true } 
        }
    )
);
// "{"y","stringify"}"
```

#### 8.对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。
深拷贝最简单粗暴的方式就是序列化：JSON.parse(JSON.stringify())，这个方式实现深拷贝会因为序列化的诸多特性导致诸多的坑点：比如下面循环引用问题。
```js
// 对包含循环引用的对象（对象之间相互引用，形成无限循环）执行此方法，会抛出错误。 
const obj = {
  name: "loopObj"
};
const loopObj = {
  obj
};
// 对象之间形成循环引用，形成闭环
obj.loopObj = loopObj;
function deepClone(obj) {
  return JSON.parse(JSON.stringify(obj));
}
deepClone(obj)
/**
 VM44:9 Uncaught TypeError: Converting circular structure to JSON
    --> starting at object with constructor 'Object'
    |     property 'loopObj' -> object with constructor 'Object'
    --- property 'obj' closes the circle
    at JSON.stringify (<anonymous>)
    at deepClone (<anonymous>:9:26)
    at <anonymous>:11:13
 */
```


#### 9.所有以 symbol 为属性键的属性都会被完全忽略掉，即便 replacer 参数中强制指定包含了它们

```js
JSON.stringify({ [Symbol.for("json")]: "stringify" }, function(k, v) {
    if (typeof k === "symbol") {
      return v;
    }
  })

// undefined
```

关于 replacer 是什么呢，它是 JSON.stringify() 的第二个参数，我们比较少的会用到，所以很多时候我们会忘记 JSON.stringify() 第二个、第三个参数，场景不多，但是用的好的话会非常的方便，关于 JSON.stringify() 第九大特性的例子中对 replacer 参数不明白的同学先别急，其实很简单，我们马上就会在下面的学习中弄懂。

### JSON.stringify() 第二个参数

第二个参数replacer 非常强大， replacer 作为函数时，可以打破九大特性的大多数特性，直接看代码吧

```js
const data = {
  a: "aaa",
  b: undefined,
  c: Symbol("dd"),
  fn: function() {
    return true;
  }
};
// 不用 replacer 参数时
JSON.stringify(data); 

// "{"a":"aaa"}"
// 使用 replacer 参数作为函数时
JSON.stringify(data, (key, value) => {
  switch (true) {
    case typeof value === "undefined":
      return "undefined";
    case typeof value === "symbol":
      return value.toString();
    case typeof value === "function":
      return value.toString();
    default:
      break;
  }
  return value;
})
// "{"a":"aaa","b":"undefined","c":"Symbol(dd)","fn":"function() {\n    return true;\n  }"}"
```

需要注意的是，replacer 被传入的函数时，第一个传进 replacer 函数的值，不是对象的第一个键值对，而是空字符串作为 key 值，value 值是整个对象的键值对：

```js
const data = {
  a: 2,
  b: 3,
  c: 4,
  d: 5
};
JSON.stringify(data, (key, value) => {
  console.log(JSON.stringify(key)+'key');
  console.log(JSON.stringify(value)+'value');
  return value;
})
// 第一个被传入 replacer 函数的是 {"":{a: 2, b: 3, c: 4, d: 5}}
// "" key
// {"a":2,"b":3,"c":4,"d":5} value
// "a" key
// 2 value
// "b" key
// 3 value
// "c" key
// 4 value
// "d" key
// 5 value
// '{"a":2,"b":3,"c":4,"d":5}'
```

replacer 作为数组时，数组的值就代表了保留在 JSON 字符串的属性名

```js
const jsonObj = {
  name: "JSON.stringify",
  params: "obj,replacer,space"
};

// 只保留 params 属性的值
JSON.stringify(jsonObj, ["params"]);
// "{"params":"obj,replacer,space"}" 
```

如果属性名不存在，就被忽略

```js
const jsonObj = {
  name: "JSON.stringify",
  params: "obj,replacer,space"
};

// other 属性不存在，就直接忽略
JSON.stringify(jsonObj, ["params","other"]);
// "{"params":"obj,replacer,space"}" 
```

### JSON.stringify() 第三个参数 space

space 参数用来控制结果字符串里面的间距。首先看一个例子就是到这东西到底是干啥用的：

```js
const tiedan = {
  name: "弹铁蛋同学",
  describe: "今天在学 JSON.stringify()",
  emotion: "like shit"
};
JSON.stringify(tiedan, null, "🐷");
// 接下来是输出结果
// "{
// 🐷"name": "弹铁蛋同学",
// 🐷"describe": "今天在学 JSON.stringify()",
// 🐷"emotion": "like shit"
// }"
JSON.stringify(tiedan, null, 2);
// "{
//   "name": "弹铁蛋同学",
//   "describe": "今天在学 JSON.stringify()",
//   "emotion": "like shit"
// }"
```

上面代码一眼就能看出第三个参数的作用了，花里胡哨的，其实这个参数还是比较鸡肋的，除了好看没啥特别的用处。用 \t、 \n 等缩进能让输出更加格式化，更适于观看。

如果是一个数字, 则在字符串化时每一级别会比上一级别缩进多这个数字值的空格（最多10个空格）；

如果是一个字符串，则每一级别会比上一级别多缩进该字符串（或该字符串的前10个字符）。
