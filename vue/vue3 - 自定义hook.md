这一点跟 Composition API 的复用如出一辙。比如实现一个鼠标点击，输出页面坐标的 hook。

useMouse.js
```javascript
import { ref, onMounted, onUnmounted } from "vue";

export default function useMouse(){
  let pointX=ref(0)
  let pointY=ref(0)

  //这里需要使用函数，如果只使用()=>{}，无法 remove 事件，因为两个()=>{}不是同一个函数
  function getMousePoint(e){
	  	pointX.value = e.pageX;
    	pointY.value = e.pageY;

  }
  onMounted(()=>{
    window.addEventListener('click',getMousePoint)
  })

	
  onUnmounted(()=>{
window.removeEventListener("click", getMousePoint);
})

  return {pointX, pointY}
}
```

App.vue
```html
<script setup>
import useMouse from './useMouse'

let {pointX,pointY}=useMouse()
</script>

<template>
  <p>x:{{pointX}}</p>
  <p>y:{{pointY}}</p>
</template>
```