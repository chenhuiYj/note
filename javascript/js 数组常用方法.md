JS 数组常用 10 个 API（用途与示例）

> 说明：示例尽量展示返回值、是否改变原数组，以及常见用法。

---

### 1) push
- **用途**: 在数组末尾添加元素，返回新长度；会改变原数组。
- **签名**: `arr.push(...items): number`
```js
const nums = [1, 2];
const len = nums.push(3, 4);
// len === 4
// nums === [1, 2, 3, 4]  （原数组被修改）
```

### 2) pop
- **用途**: 删除并返回数组最后一个元素；会改变原数组。
- **签名**: `arr.pop(): any | undefined`
```js
const stack = [10, 20, 30];
const last = stack.pop();
// last === 30
// stack === [10, 20]
```

### 3) shift
- **用途**: 删除并返回数组第一个元素；会改变原数组。
- **签名**: `arr.shift(): any | undefined`
```js
const queue = ['a', 'b', 'c'];
const first = queue.shift();
// first === 'a'
// queue === ['b', 'c']
```

### 4) unshift
- **用途**: 在数组开头添加元素，返回新长度；会改变原数组。
- **签名**: `arr.unshift(...items): number`
```js
const list = [3, 4];
const len2 = list.unshift(1, 2);
// len2 === 4
// list === [1, 2, 3, 4]
```

### 5) slice
- **用途**: 按区间返回浅拷贝新数组，不包含结束索引；不改变原数组。
- **签名**: `arr.slice(start?, end?): any[]`
```js
const arr = [0, 1, 2, 3, 4];
const part = arr.slice(1, 4);
// part === [1, 2, 3]
// arr 不变 === [0, 1, 2, 3, 4]

// 复制整个数组
const copy = arr.slice();
// copy === [0, 1, 2, 3, 4]
```

### 6) splice
- **用途**: 在任意位置增删改元素，返回被删除的元素；会改变原数组。
- **签名**: `arr.splice(start, deleteCount?, ...items): any[]`
```js
const arr2 = [1, 2, 3, 4];
// 删除：从索引 1 开始删除 2 个
const removed = arr2.splice(1, 2);
// removed === [2, 3]
// arr2 === [1, 4]

// 插入：在索引 1 处插入 'a', 'b'
arr2.splice(1, 0, 'a', 'b');
// arr2 === [1, 'a', 'b', 4]

// 替换：从索引 2 开始，删 1 个，插入 9
arr2.splice(2, 1, 9);
// arr2 === [1, 'a', 9, 4]
```

### 7) map
- **用途**: 基于每项映射生成等长新数组；不改变原数组。
- **签名**: `arr.map((item, index, array) => newItem): any[]`
```js
const prices = [10, 20, 30];
const withTax = prices.map(p => p * 1.1);
// withTax === [11, 22, 33]
// prices 不变 === [10, 20, 30]
```

### 8) filter
- **用途**: 过滤出满足条件的元素组成新数组；不改变原数组。
- **签名**: `arr.filter((item, index, array) => boolean): any[]`
```js
const users = [
	{ id: 1, active: true },
	{ id: 2, active: false },
	{ id: 3, active: true }
];
const actives = users.filter(u => u.active);
// actives === [{id:1,active:true},{id:3,active:true}]
```

### 9) reduce
- **用途**: 将数组归纳为一个值（数字、对象等）；不改变原数组。
- **签名**: `arr.reduce((acc, cur, index, array) => nextAcc, initialValue?)`
```js
// 累加求和
const nums2 = [1, 2, 3, 4];
const sum = nums2.reduce((acc, n) => acc + n, 0);
// sum === 10

// 按字段分组
const people = [
	{ name: 'A', group: 'x' },
	{ name: 'B', group: 'y' },
	{ name: 'C', group: 'x' }
];
const grouped = people.reduce((acc, p) => {
	(acc[p.group] ||= []).push(p);
	return acc;
}, {});
// grouped === { x: [{A},{C}], y: [{B}] }
```

### 10) find
- **用途**: 返回第一个满足条件的元素；找不到返回 `undefined`；不改变原数组。
- **签名**: `arr.find((item, index, array) => boolean): any | undefined`
```js
const inventory = [
	{ sku: 'A1', qty: 0 },
	{ sku: 'B2', qty: 3 },
	{ sku: 'C3', qty: 5 }
];
const firstInStock = inventory.find(p => p.qty > 0);
// firstInStock === { sku: 'B2', qty: 3 }
```

---

补充说明：
- **是否变更原数组**：`push/pop/shift/unshift/splice/sort/reverse` 会改变原数组；`slice/map/filter/reduce/find/some/every/includes/concat` 不改变原数组。
- **遍历优先选 map/filter/reduce**：语义更清晰、可链式调用；需要副作用时再考虑 `forEach`。


