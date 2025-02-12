笔记摘录文章：[前端工程师的自我修养-关于 Babel 那些事儿](https://juejin.im/post/6844904079118827533)

### 1.什么是babel

直接点说就是， Babel 是一个 JavaScript 编译器，用于将 ECMAScript 2015+（ES6） 版本的代码转换为向下兼容的 JavaScript(ES5) 语法，以便能够运行在当前版本和旧版本的浏览器或其他环境中。

### 2.babel的原理

babel的工作原理的核心就是 AST (抽象语法树)。先将ECMAScript 2015+（ES6） 版本源码转成抽象语法树，然后对语法树进行处理，生成新的语法树，最后根据新语法树生成新的 JavaScript(ES5) 代码，整个编译过程可以分为 3 个阶段 parsing (解析)、transforming (转换)、generating (生成)。

![image](https://user-images.githubusercontent.com/27403818/90385488-d0806800-e0b5-11ea-9942-153721904aa4.png)

Babel 只负责编译新标准引入的新语法，比如 Arrow function、Class、ES Module 等，它不会编译原生对象新引入的方法和 API，比如 Array.includes，Map，Set 等，这些需要通过 Polyfill（polyfill 的翻译过来就是垫片，垫片就是垫平不同浏览器环境的差异） 来解决