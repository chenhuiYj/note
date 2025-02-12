#### 遵循规范

require 是 CommonJS 规范引入方式

import 是 es6 的一个语法标准，如果要兼容浏览器的话必须转化成es5的语法

#### 调用时间

require是运行时调用，所以require理论上可以运用在代码的任何地方（虽然这么说但是还是一般放开头）

import是编译时调用，所以必须放在文件开头或者 <script></script> 标签的开始的地方

#### 本质

require是赋值过程，其实require的结果就是对象、数字、字符串、函数等，再把require的结果赋值给某个变量

import是解构过程，传的是值引用。

PS：目前 import 无论在 node 和浏览器上都不能直接使用吧，需要  babel 编译
import 会被转成 require  

#### 写法

require 遵循 CommonJS/AMD，只能在运行时确定模块的依赖关系及输入/输出的变量，无法进行静态优化。
用法只有以下三种简单的写法：
```js
const fs = require('fs')
exports.fs = fs
module.exports = fs
```

import 遵循 ES6 规范，支持编译时静态分析，便于JS引入宏和类型检验。动态绑定。
写法就比较多种多样：
```js
import fs from 'fs'
import {default as fs} from 'fs'
import * as fs from 'fs'
import {readFile} from 'fs'
import {readFile as read} from 'fs'
import fs, {readFile} from 'fs'

export default fs
export const fs
export function readFile
export {readFile, read}
export * from 'fs'
```


#### 小结

1. 通过 require 引入基础数据类型时，属于复制该变量。
2. 通过 require 引入复杂数据类型时，数据浅拷贝该对象。
3. 出现模块之间的循环引用时，会输出已经执行的模块，而未执行的模块不输出（比较复杂）
4. CommonJS 模块默认 export 的是一个对象，即使导出的是基础数据类型