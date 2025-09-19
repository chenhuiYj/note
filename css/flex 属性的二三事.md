### 1.flex 是哪几个属性的简写

flex 属性用于设置或检索弹性盒模型对象的子元素如何分配空间，是 flex-grow、flex-shrink 和 flex-basis 属性的简写属性。默认值为0 1 auto。后两个属性可选。

> 如果元素不是弹性盒模型对象的子元素，则 flex 属性不起作用

### 2.每一个属性代表什么意思，具体例子

为了方便理解，下面先从 flex-basis 说起

#### 2-1.flex-basis

flex-basis 属性用于设置弹性盒伸缩基准值，可以设置具体宽度或者百分比，默认值是 auto。

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>demo</title>
<style> 
.main
{
	width:240px;
	height:300px;
	border:1px solid black;
	display:flex;
}

.item
{
	flex:1 1 auto;
}	
</style>
</head>
<body>

<div class="main">
  <div style="background-color:coral;" class="item red">红</div>
  <div style="background-color:lightblue;" class="item blue">蓝</div>  
  <div style="background-color:lightgreen;" class="item green">绿</div>
</div>

</body>
</html>
```

![image](https://user-images.githubusercontent.com/27403818/93066555-b8980600-f6ac-11ea-9e0f-40dbd90cec3c.png)

比如上面，由于只设置了 flex:1 1 auto; 那么 flex-basis 实际上就是 auto ，那么每个子元素的宽度实际上就父元素的宽度/子元素的数量。就是 240/3 。每一个子元素的宽度就是 80px。


flex-basis 可以设置一个具体的宽度，也可以是一个百分比。比如把红色的 div，设置成 flex-basis:40px; 


![image](https://user-images.githubusercontent.com/27403818/93157314-8df58e00-f73c-11ea-8a95-e258f2122dcd.png)

效果如图，至于为什么红色是 96px，下面解释。

#### 2-2.flex-grow

> 用来“瓜分”父项的“剩余空间”

如果子元素总的宽度小于父元素，就会瓜分

上面的结果，为什么红色是 96px 呢？首先把 flex-grow 设置 0，看下每一个元素的初始大小是多少。

![image](https://user-images.githubusercontent.com/27403818/93157950-cba6e680-f73d-11ea-84e4-47358a18e22b.png)

可以看到，红色的由于设置了 flex-basis:40px; 所以就是 40px。蓝色和绿色只有一个字体的宽度，就是 16px; 

由于子元素的宽度就是 40+16+16=72 ，小于父元素的 240。如果把 flex-grow 设置 1，那么父元素的剩余宽度 （240-72=168）就会平均分配给子元素 （168/3=56）。所以最后子元素的宽度是：红色=40+56=96px  蓝色=16+56=72px 绿色=16+56=72px。这就解释了，为什么 红色设置了 flex-basis:40px; 最后得出宽度是 96px;

如果给其中一个元素，比如蓝色的 flex-grow 设置 2 ，那么蓝色的子元素，瓜分剩余父元素的宽度就是红色或者绿色的两倍。（也可以理解为，把父元素的剩余宽度分成四份 168/4=42 ，蓝色子元素占其中两份，红色和绿色各占一份。因为红蓝绿三个子元素的 flex-grow 分别是 1:2:1。加起来就是 4 ）。最后算出来的宽度就是：蓝色=16+42*2=100，红色=40+42=82，绿色 16+42=58

![image](https://user-images.githubusercontent.com/27403818/93160792-ae751680-f743-11ea-826a-53f25a05e426.png)

#### 2-3.flex-shrink

> 用来“吸收”超出的空间

既然知道了 flex-grow 是用来瓜分剩余的父元素的。想必大家都想到了，如果子元素的总宽度大于父元素呢，这时候就需要 flex-shrink 来对子元素进行相应的缩小了

比如把 子元素的 flex-basis 都改成 100px;

```
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>demo</title>
<style> 
.main
{
	width:240px;
	height:300px;
	border:1px solid black;
	display:flex;
}

.item
{
	flex:1 1 100px;
}
</style>
</head>
<body>

<div class="main">
  <div style="background-color:coral;" class="item red">红</div>
  <div style="background-color:lightblue;" class="item blue">蓝</div>  
  <div style="background-color:lightgreen;" class="item green">绿</div>
</div>

</body>
</html>
```
![image](https://user-images.githubusercontent.com/27403818/93162238-e893e780-f746-11ea-8cdd-9d622961960d.png)

但是最终每个子元素最后的宽度都是 80px ，这个结果的计算方式也非常简单。

首先三个子元素的 flex-basis 都是 100 ，那么子元素的总宽度就是 300，比父元素的宽度（240）多了 60。那么这 60 就需要子元素“消化掉”，由于设置了 flex:1 1 100px; 子元素的 flex-shrink 实际上都是 1 。那么就是把 60 分成 3份，每一份 20，那么子元素的实际宽度就是：预设宽度（100） - “消化掉的宽度” （20）= 80，所以每一个子元素都是 80 。

如果把其中一个子元素，比如绿色的子元素 flex-shrink 设置为 2 。其实这个和 flex-grow 类似，把多出来的 60 分成四份给子元素“消化掉”，每一份就是 15，然后 绿色的占两份，红色和蓝色各占一份（因为红蓝绿三个子元素的 flex-shrink 分别是 1:1:2。加起来就是 4）。那么最后计算出来的宽度就是：红色=100-15=85，蓝色=100-15=85，绿色：100-15*2=70

![image](https://user-images.githubusercontent.com/27403818/93176581-0cb1f180-f764-11ea-8ae8-c4a83b0830bc.png)

> 1.关于宽度计算，是根据 flex-grow 还是 flex-shrink。只要关注子元素的总宽度和父元素宽度对比就行了。如果父元素宽度足够：flex-grow有效，flex-shrink无效。如果父元素宽度不够：flex-shrink 有效，flex-grow无效。
>
> 2.注意，设为Flex布局以后，子元素的float、clear和vertical-align属性将失效。

### 3.其他例子


#### 3-1. flex:0 0 1;

#### 3-2. flex:1;

#### 3-3. flex auto auto 1;