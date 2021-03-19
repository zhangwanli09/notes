# 圣杯，双飞翼布局

两者都是三栏布局的优化解决方案。

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

特殊情况下有问题，当窗口无限变窄，布局会混乱。当 main 部分的宽小于 left 部分时就会发生布局混乱。

### 双飞翼

