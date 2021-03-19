# 圣杯，双飞翼布局

两者都是`三栏布局`的优化解决方案。不过有点过度优化了，现在大多使用`flex`布局。

![示例](../imgs/img8.png ':size=500')

常规情况下，布局从上到下，从左到右。如下：

```html
<header>header</header>
<section>
  <aside>left</aside>
  <section>main</section>
  <aside>right</aside>
</section>
<footer>footer</footer>
```

这样是没问题的，但是，如果想让中间的`main`部分优先显示的话，是可以做`布局优化`的。

因为浏览器渲染引擎在构建和渲染渲染树是异步的（谁先构建好谁先显示），那么将`<section>main</section>`部分提前即可优先渲染。

### 圣杯

圣杯布局，通过`css`配合`DOM`结构，优化DOM渲染。

```html
<header>header</header>
<section class="wrapper">
  <section class="col main">main</section>
  <aside class="col left">left</aside>
  <aside class="col right">right</aside>
</section>
<footer>footer</footer>

<style>
header, footer {height: 50px; background-color: #999;}
.wrapper {padding: 0 100px; overflow: hidden;}
.col {float: left; position: relative;}
.main {width: 100%; height: 200px; background-color: yellow;}
.left {width: 100px; height: 200px; margin-left: -100%; left: -100px; background-color: red;}
.right {width: 100px; height: 200px; margin-left: -100px; right: -100px; background-color: blue;}
</style>
```

特殊情况下`有问题`，当窗口无限变窄，布局会混乱。当 main 部分的宽小于 left 部分时就会发生布局混乱。

### 双飞翼

淘宝针对圣杯的缺点做了`优化`。

```html
<header>header</header>
<section class="wrapper">
  <section class="col main">
    <section class="main-wrap">main</section>
  </section>
  <aside class="col left">left</aside>
  <aside class="col right">right</aside>
</section>
<footer>footer</footer>

<style>
header, footer {height: 50px; background-color: #999;}
.wrapper {padding: 0; overflow: hidden;}
.col {float: left;}
.main {width: 100%; background-color: yellow;}
.main-wrap {margin: 0 100px 0; height: 200px;}
.left {width: 100px; height: 200px; margin-left: -100%; background-color: red;}
.right {width: 100px; height: 200px; margin-left: -100px; background-color: blue;}
</style>
```

同样使用了`float`和 负值`margin`，不同的是，并没有使用 relative 相对定位，而是增加了 dom 结构，增加了一个层级。解决了圣杯布局的缺陷。

### 绝对定位

虽然绝对定位也可以实现三栏布局，但是`高度不可控`，两侧高度无法支撑总高度。

```html
<header>header</header>
<section class="wrapper">
  <section class="col main">main</section>
  <aside class="col left">left</aside>
  <aside class="col right">right</aside>
</section>
<footer>footer</footer>

<style>
header, footer {height: 50px; background-color: #999;}
.wrapper {position: relative;}
.main {height: 200px; margin: 0 100px; background-color: yellow;}
.left, .right {width: 100px; height: 200px; position: absolute; top: 0;}
.left {left: 0; background-color: red;}
.right {right: 0; background-color: blue;}
```

### 总结

我还是选择`flex`。

> 参考：https://juejin.cn/post/6844903510933258247
