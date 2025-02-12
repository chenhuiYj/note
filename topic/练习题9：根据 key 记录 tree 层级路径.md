比如给出一个数据

```
{
    key:1,
  children:[{
     key:2,
     children:null
  }]
}
```

需要根据key记录每一项对应的层级路径，如上面的数据。需要返回` {1:'1',2:'1-2'}`

实现这个思路本质上就是写一个递归，只是多了一个，每次传入的不止是数据，还有层级路径


demo 数据
```
let arr=[
{
    key:1,
    children:[
        {
            key:2,
            children:[{
                key:3,
                children:null

            }]
        },
        {
            key:6,
            children:[{
                key:7,
                children:null
            
            }]
        }
    ]

},
{
    key:'4',
    children:[{
        key:'5',
        children:null
        
    }]
}]
```

实现代码

```
function getPaths(list){
    let result={}
    function handle(data,path){
        for(let item of data){
            result[item.key]=path?path+'-'+item.key:path+item.key
            if(item.children){
                handle(item.children,result[item.key])
            } 
        }
    }
    handle(list,'')
    return result
}
```

测试结果

```
let keys=getPaths(arr)
console.log(keys)

// {1: "1", 2: "1-2", 3: "1-2-3", 4: "4", 5: "4-5", 6: "1-6", 7: "1-6-7"}
```

