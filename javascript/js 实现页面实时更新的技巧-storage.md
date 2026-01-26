提到 storage ，想必大家第一反应是 localStorage 和 sessionStorage。或者是两者的 API, getItem、setItem 和 removeItem 。但今天主要讲述的是 storage 事件。

### Storage 事件

顾名思义，就是 Storage 数据发生变化时，会触发 storage 事件，可以指定这个事件的监听函数。

```js
window.addEventListener('storage', handleStorageChange);
```

事件监听函数如其他事件函数一般，接受一个event实例对象作为参数。这个实例对象继承了 StorageEvent 接口，有几个特有的属性，都是只读属性。


|属性|解析|
|----|----|
|StorageEvent.key|字符串，表示发生变动的键名。如果 storage 事件是由clear()方法引起，该属性返回null。|
|StorageEvent.newValue|字符串，表示新的键值。如果 storage 事件是由clear()方法或删除该键值对引发的，该属性返回null。|
|StorageEvent.oldValue|字符串，表示旧的键值。如果该键值对是新增的，该属性返回null。|
|StorageEvent.storageArea|对象，返回键值对所在的整个对象。也说是说，可以从这个属性上面拿到当前域名储存的所有键值对。|
StorageEvent.url|字符串，表示原始触发 storage 事件的那个网页的网址。|

```js
<!DOCTYPE html>
<html>
    <head lang="en">
        <title>test storage</title>
    </head>
    <body>
        <script>
            localStorage.setItem("test", "hello 守候");
            window.addEventListener("setItemEvent", function(e) {
                alert(e.newValue);
            });
            
        </script>
    </body>
</html>
```
同时打开多个页面，当其中的一个页面导致储存的数据发生改变时，只有在其他页面才能观察到监听函数的执行。可以通过这种事件机制，实现多个页面之间的实时通信。

**注：**

如果浏览器只打开一个窗口，可能观察不到这个事件。

如果非要在同一个网页中监听该事件，需要重写localStorage的方法, 抛出自定义的事件。（非常不推荐，会污染原生属性）

```js
<!DOCTYPE html>
<html>
    <head lang="en">
        <title>A</title>
    </head>
    <body>
        <script>
            var orignalSetItem = localStorage.setItem;
            localStorage.setItem = function(key, newValue) {
                var setItemEvent = new Event("setItemEvent");
                setItemEvent.newValue = newValue;
                window.dispatchEvent(setItemEvent);
                orignalSetItem.apply(this, arguments);
            }
            window.addEventListener("setItemEvent", function(e) {
                alert(e.newValue);
            });
            localStorage.setItem("nm", "1234");
</script>
    </body>
</html>
```


