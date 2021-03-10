# 对BFC规范的理解

块格式化上下文（BFC）Block Formatting Context是web页面的可视css渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。

下列方式会创建BFC：
1. 跟元素（html）
2. 浮动元素（float）
3. 绝对定位元素（position为absolute或fixed）
4. 行内块元素（display为inline-block）
5. 表格布局元素（display为table）
6. 弹性布局元素（display为flex）

BFC主要作用：
1. 清除浮动
2. 防止同一BFC容器中的相邻元素的外边距重叠问题
