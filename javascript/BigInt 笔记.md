BigInt 是 es10 中提供的一个新的数据类型，与String,Number, Boolean, Null, Undefined, Symbol相同，都是基本类型。

在JavaScript中，Number可以准确表达的最大数字是2^53，没有比这个更大的数字。如果数字比这个数字更大，就无法进行在准确的输出。

比如

```
Number('12630717197566440063')
// 输出：12630717197566440000
```

但是用 BigInt 就可以精确的输出

```
BigInt('12630717197566440063')
// 输出：12630717197566440063n
```

可以看到，BigInt 类型就是在数字后面加了一个n。

用 typeof 测试也是输出 bigint

```
typeof 123n; // → 'bigint'
```

### 注意事项

1.不能使用Math对象中的方法

2.不能与Number类型进行混合运算

3.与Number类型宽松相等
```
0n === 0
//false

0n == 0
//true
```

4.Number 和 BigInt 可以进行比较，也可以进行排序

```
2n > 2
//false

2n >= 2
//true
```

5.在 JSON.stringify 中使用是引发报错

6.不能带有小数点，相关运算如果是小数会被取整

```
BigInt('123.56') // SyntaxError: Cannot convert 123.56 to a BigInt

117n/30n // 3n
```