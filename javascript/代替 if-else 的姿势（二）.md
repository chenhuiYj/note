### 1.范围查询
比如判一个值是否在9-12或者14-18里面
之前做法是
```
let num1= 15
let num2= 13
if((num1>=9&&num1<=12)||(num1>=14&&num1<=18)){
     // 数字在范围内
}
if((num2>=9&&num2<=12)||(num2>=14&&num2<=18)){
     // 数字在范围内
}
```
现在可以用some 进行封装一下

```
function handleCheckRange(num, ...ranges){
    return ranges.some(item=>num>=item[0]&&num<=item[1])
}
handleCheckRange(num1,[9,12],[14,18]) //true
handleCheckRange(num2,[9,12],[14,18]) //false
```

这样写貌似没什么问题，如果以后时间变了，或者又有其他需求，还是要这样写一下，
### 2.多条件查询

相信大家都会遇到可能会有多个条件组合的问题

比如有一个参与热卖活动赠积分活动，活动状态(status)，预热中（status=1），进行中（status=2）。用户类型（type）也有分普通用户（type=1）vip用户（type=2）

规则是：
1.在预热中参与活动，vip用户赠送1000积分，普通用户赠送700积分。
1.在进行中参与活动，vip用户赠送800积分，普通用户赠送300积分。

之前写法是
```
let status=1
let type=2
if(status===1){
     if(type===1){
           console.log('普通用户在预售中参与活动，赠送700积分')
    }
    else if(type===2){
           console.log('vip用户在预售中参与活动，赠送1000积分')
    }
}
else if(status===2){
     if(type===1){
           console.log('普通用户在进行中参与活动，赠送300积分')
    }
    else if(type===2){
           console.log('vip用户在进行中参与活动，赠送800积分')
    }
}

// 或者

if(status===1&&type===1){
    console.log('普通用户在预售中参与活动，赠送700积分')
}
else if(status===1&&type===2){
    console.log('vip用户在预售中参与活动，赠送1000积分')
}
else if(status===2&&type===1){
    console.log('普通用户在进行中参与活动，赠送300积分')
}
else if(status===2&&type===2){
    console.log('vip用户在进行中参与活动，赠送800积分')
}
```

但是现在可以使用 obj写法

```
let obj={
   'status=1&type=1':'普通用户在预售中参与活动，赠送700积分',
   'status=1&type=2':'vip用户在预售中参与活动，赠送1000积分',
   'status=2&type=1':'普通用户在进行中参与活动，赠送300积分',
   'status=2&type=2':'普通用户在进行中参与活动，赠送800积分'
}
```

console.log(obj[`status=${status}&type=${type}`])

### 多个或条件

比如输如一个城市名字，输出一个省份名称

```
let city='广州'
if(city==='广州'||city==='佛山'){
    console.log('广东')
}
else if(city==='海口'||city==='三亚'){
    console.log('海南')
}
```


这样写的弊端就是，如果if-else 数量一多，代码就会多，而且判断的条件会变得很长，还有一个问题就是风格有可能不会统一

下面用其他方法进行优化下
方法一
```
let arr=[
    {
        city:'广州',
        province:'广东'
    },
    {
        city:'佛山',
        province:'广东'
    },
    {
        city:'海口',
        province:'海南'
    },
    {
        city:'三亚',
        province:'海南'
    }
]
console.log(arr.filter(item=>item.city===city)[0].province)//广东
```
方法二
```
方式一
let city='广州'
let obj={
    '广州':'广东',
    '佛山':'广东',
    '海口':'海南',
    '三亚':'海南',
}
console.log(obj[city])// 广东

// 方式二------------------------------------
let obj={
    '广州,佛山':'广东',
    '海口,三亚':'海南',
}
for(let key in obj){
    if(key.includes(city)){
        console.log(obj[key])
        break
    }
}
//广东


//这个方法是有bug的，比如
city='州'
for(let key in obj){
    if(key.includes(city)){
        console.log(obj[key])
        break
    }
}
// 广东
// 解决这个问题很简单，可以使用Map
city='广州'
let map=new Map([
     [['广州','佛山'],'广东'],
     [['海口','三亚'],'海南'],
])
for (let key of map.keys()) {
  if(key.includes(city)){
        console.log(obj[key])
        break
    }
}
//广东
```

#### 参考链接
[[浅析]特定场景下取代if-else和switch的方案](https://juejin.im/post/5b4b73e7f265da0f96287f0a)
[JavaScript 复杂判断的更优雅写法](https://juejin.im/post/5bdfef86e51d453bf8051bf8)