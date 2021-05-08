# 着色器语言：GLSL ES

## 顶点着色器（Vertex shader）

> 描述顶点的特征，如位置、颜色等。<br>
> 顶点着色器里的顶点就是补间动画里的关键帧<br>
> 补间动画是在两个关键帧(非连续帧)之间，以某种算法自动计算物体运动的插值，从而形成一种过渡效果。<br>
顶点着色程序，要写在type=“x-shader/x-vertex” 的script中

```html
//顶点着色器
<script id="vertexShader" type="x-shader/x-vertex">
  void main() {
    <!-- 前三个参数是x、y、z，第4个参数默认1.0 -->
    gl_Position = vec4(0.0, 0.0, 0.0, 1.0);
    gl_PointSize = 100.0;
  }
</script>
<!-- void main() {…… } 是主体函数。 -->
<!-- 在顶点着色器中，gl_Position 是顶点的位置，gl_PointSize 是顶点的尺寸，这种名称都是固定的，不能写成别的。 -->
```

### attribute

```js
gl_Position = vec4(0,0,0,1);
```

这是一种将数据写死了的硬编码，缺乏可扩展性。
我们要让这个点位可以动态改变，那就得把它变成attribute变量。

> attribute 变量是只有顶点着色器才能使用它的。<br>
> js 可以通过attribute 变量向顶点着色器传递与顶点相关的数据。

js向attribute 变量传参的步骤

```html
<!-- 1、在顶点着色器中声明attribute 变量。 -->
<script id="vertexShader" type="x-shader/x-vertex">
  // attribute 是存储限定符，是专门用于向外部导出与点位相关的对象的，这类似于es6模板语法中export 。
  // vec4 是变量类型，vec4是4维矢量对象。
  // a_Position 是变量名，之后在js中会根据这个变量名导入变量。这个变量名是一个指针，指向实际数据的存储位置。
  attribute vec4 a_Position;
  void main(){
    gl_Position = a_Position;
    gl_PointSize = 50.0;
  }
</script>
<script>
  <!-- 2、在js中获取attribute 变量 -->
  // gl.getAttribLocation() 是获取着色器中attribute 变量的方法。
  // 因为着色器和js 是两个不同的语种，着色器无法通过window.a_Position 原理向全局暴露变量。
  // 那我们要在js 里获取着色器暴露的变量，就需要找人来翻译，这个人就是程序对象。
  // gl.program 是初始化着色器时，在上下文对象上挂载的程序对象。
  // 'a_Position' 是着色器暴露出的变量名。
  const a_Position=gl.getAttribLocation(gl.program,'a_Position');
  <!-- 3、修改attribute 变量   -->
  // attribute 变量即使在js中获取了，他也是一个只会说GLSL ES语言的人，他不认识js 语言，所以我们不能用js 的语法来修改attribute 变量的值：a_Position.a = 3
  // 需要用gl.vertexAttrib3f() 给 attribute 变量赋值
  // a_Position 就是咱们之前获取的着色器变量。
  // 后面的3个参数是顶点的x、y、z位置
  gl.vertexAttrib3f(a_Position,0.0,0.5,0.0);
  <!-- 4、修改位置后，绘制最新图   -->
  gl.clearColor(0.0, 0.0, 0.0, 1.0);
  gl.clear(gl.COLOR_BUFFER_BIT);
  gl.drawArrays(gl.POINTS, 0, 1);
</script>
```

## 片元着色器（Fragment shader）

> 进行逐片元处理，如光照。<br>
> 片元着色器里的片元就是关键帧之间以某种算法算出的插值，webgl里的片元是像素的意思。
片元着色程序，要写在type=“x-shader/x-fragment” 的script中

```html
//片元着色器
<script id="fragmentShader" type="x-shader/x-fragment">
  void main() {
    <!-- 参数是r,g,b,a -->
    gl_FragColor = vec4(1.0, 1.0, 0.0, 1.0);
  }
</script>
<!-- gl_FragColor 是片元的颜色 -->
```

**两点决定一条直线大家知道不？顶点着色器里的顶点就是决定这一条直线的两个点，片元着色器里的片元就是把直线画到画布上后，这两个点之间构成直线的每个像素。**

## webgl 函数的命名规律

GLSL ES里函数的命名结构是：<基础函数名><参数个数><参数类型>
以vertexAttrib3f(location,v0,v1,v2,v3) 为例：

- vertexAttrib：基础函数名
- 3：参数个数，这里的参数个数是要传给变量的参数个数，而不是当前函数的参数个数
- f：参数类型，f 代表float 浮点类型，除此之外还有i 代表整型，v代表数字……

#### vertexAttrib3f()的同族函数

gl.vertexAttrib3f(location,v0,v1,v2) 方法是一系列修改着色器中的attribute 变量的方法之一，它还有许多同族方法，如：

```js
gl.vertexAttrib1f(location,v0) 
gl.vertexAttrib2f(location,v0,v1)
gl.vertexAttrib3f(location,v0,v1,v2)
gl.vertexAttrib4f(location,v0,v1,v2,v3)
```

它们都可以改变attribute 变量的前n 个值。

比如 vertexAttrib1f() 方法自定一个矢量对象的v0值，v1、v2 则**默认为0.0**，v3**默认为1.0**，其数值类型为float 浮点型

## 用鼠标控制点位

#### 用向量法求出鼠标在canvas 2d 中的坐标（cssX, cssY）,即下图的向量c

![](./assets/向量法.png)
向量a(clientX,clientY)，向量c(left,top)
向量c = a-c = (clientX-left,clientY-top)

将 向量c 视之为坐标点c，那点c 就是鼠标在canvas 画布中的css 位。

因为html 坐标系中的坐标原点和轴向与canvas 2d是一致的，所以在我们没有用css 改变画布大小，也没有对其坐标系做变换的情况下，鼠标点在canvas 画布中的css 位就是鼠标点在canvas 2d坐标系中的位置。

#### 将canvas坐标转为webgl坐标

1、解决坐标原点位置的差异。

```javascript
const [halfWidth,halfHeight]=[width/2,height/2];
const [xBaseCenter,yBaseCenter]=[cssX-halfWidth,cssY-halfHeight];
```

上面的[halfWidth,halfHeight]是canvas 画布中心的位置。
[xBaseCenter,yBaseCenter] 是用鼠标位减去canvas 画布的中心位，得到的就是鼠标基于画布中心的位置。

2.解决y 方向的差异，webgl 里的y 轴和canvas 2d 里的y轴相反。

```javascript
const yBaseCenterTop=-yBaseCenter;
```

3、解决坐标基底的差异，webgl 的设值为0-1，画布宽高比例。

```javascript
const [x,y]=[xBaseCenter/halfWidth,yBaseCenterTop/halfHeight]
```

最终代码

```javascript
canvas.addEventListener('click',function(event){
  const {clientX,clientY}=event;
  const {left,top,width,height}=canvas.getBoundingClientRect();
  const [cssX,cssY]=[
    clientX-left,
    clientY-top
  ];
  const [halfWidth,halfHeight]=[width/2,height/2];
  const [xBaseCenter,yBaseCenter]=[cssX-halfWidth,cssY-halfHeight];
  const yBaseCenterTop=-yBaseCenter;
  const [x,y]=[xBaseCenter/halfWidth,yBaseCenterTop/halfHeight];
})
```

## webgl 同步绘图原理

gl.drawArrays(gl.POINTS, 0, 1) 方法和canvas 2d 里的ctx.draw() 方法是不一样的，ctx.draw() 真的像画画一样，一层一层的覆盖图像。
**gl.drawArrays() 方法只会同步绘图，走完了js 主线程后，再次绘图时，就会从头再来。**
也就说，异步执行的drawArrays() 方法会把画布上的图像都刷掉。
