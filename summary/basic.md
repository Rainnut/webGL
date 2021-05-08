# 获取上下文

```javascript
onst canvas = document.querySelector("#canvas");
canvas.width = window.innerWidth;
canvas.height = window.innerHeight;
//二维画笔
// const gl=canvas.getContext('2d');
//三维画笔
const gl = canvas.getContext("webgl");
```

# 设置底色

```javascript
//声明颜色 rgba
gl.clearColor(1, 1, 1, 1);
//刷底色
gl.clear(gl.COLOR_BUFFER_BIT);
//获取当前底色值
gl.getParameter(gl.COLOR_CLEAR_VALUE)
```

* WebGLRenderingContext.clearColor(red=0, green=0, blue=0, alpha=0)
  用于设置清空颜色缓冲时的颜色值，指定调用 clear() 方法时使用的颜色值。
  这些值在0到1的范围间。
* WebGLRenderingContext.clear()

# webgl 坐标系

> 原点在画布中心
![](./assets/webgl坐标系.png)
