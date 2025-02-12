不使用 for, while 以及 所有数组内置的 API ，实现将一个空数组赋值成 [0,2,4,6,8....100]

实现方式
```
function handle(){
    let arr=[]
    let item=0
    function fn(){
        arr[item]=item*2
        item+=1
        if(item<=50){
            fn()
        }
    }
    fn()
    return arr
}
handle()
```
把代码贴到 console 就可以正常的输出了，但是如果下次如果要求输出 [0,2,4....200]呢？这个很好解决，只要加个参数 num 就可以实现。规定循环多少次才输出

```
function handle(num){
    let arr=[]
    let item=0
    function fn(){
        arr[item]=item*2
        item+=1
        if(item<=num){
            fn()
        }
    }
    fn()
    return arr
}
handle(50)
```