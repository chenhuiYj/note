关于 react useEffect hook 可以看做是 componentDidMount，componentDidUpdate 和 componentWillUnmount 这三个函数的组合。

```
import React, { useState, useEffect } from 'react';

function Example() {
  const [status, setStatus] = useState(false);

  useEffect(() => {
    document.title = `现在状态是：${status?:'开':'关'}`;
  });

  return (
    <div>
      <button onClick={() => useState(!status)}>
        Click me
      </button>
    </div>
  );
}
```

根据上面的代码，每次点击按钮，都可以改变页面的标题（“开”和“关”的来回切换）

但是问题也出来了，因为 useEffect 在第一次渲染后和每次更新后都会执行。如果页面上有多个 state ，如果只改变了 status ，也会引起整个页面的重新渲染。或者我只想改变 status 的时候，再 useEffect ，其他情况就不执行呢？这个实现不难，只需要加上第二个参数就好

比如想仅当 status 改变再执行，下面的写法就可以实现
```
useEffect(() => {
    document.title = `现在状态是：${status?:'开':'关'}`;
  },[status]);
```

比如想 只在第一次渲染后执行一次，下面的的写法就可以实现

```
useEffect(() => {
    document.title = `现在状态是：${status?:'开':'关'}`;
  },[]);
```

如此一来，可以避免 useEffect 多次重复的执行，避免一些无用的执行。这样可以实现执行次数上的优化。
