#### 1.data 和 computed 的区别

核心的区别在于 data 中的属性并不会随赋值变量的改动而改动，而computed 会

```
<script>
    let outer_obj = {num: 30}
    let app = new Vue({
        el: "#app",
        data: {
            temp: outer_obj, // 这一句一定要加，必须在 data 对象上存在才能让 Vue 将它转换为响应式
            num1: outer_obj.num
        },
        computed: {
            c_num () {
                return outer_obj.num
            }
        }
    })
</script>
```

当改变 outer_obj.num 的值，c_num 会随之改变，而 num1 不会

#### 2.methods 和 computed 的区别

1.computed 是基于它们的响应式依赖进行缓存的。只要 computed 依赖的那个数据不变，computed 就不会重新计算，而 methods 是只要页面有任何数据变化导致数据的更新，methods 就会重新计算

2.computed 不能带参数

#### 3.Vue 实例在渲染的时候数据解析的顺序

Vue 实例在渲染的时候数据解析的顺序问题，结论是props->methods->data->computed->watch->created

![image](https://user-images.githubusercontent.com/27403818/90228161-539c8680-de48-11ea-8910-0fffadb22d63.png)
