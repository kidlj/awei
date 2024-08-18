---
title: CSS
---

### 水平格式化

#### 基本原则

* 块级元素生成一个元素框，旁边不能有其它元素。换句话说，它在元素框之前和之后生成了“分隔符”。

* 水平格式化的“7大属性”是：margin-left, border-left, padding-left, width, padding-right, border-right 和 margin-right。这 7 个属性的值加在一起必须是其父元素的内容区宽度。

* width 值默认是指内容区的宽度，不包括 padding 和 border；可以通过设定 `box-sizing` 为 content-box 或 border-box 来更改这一特性。

#### 宽度设定

水平格式化的 7 个属性中，只有 3 个属性可以设定为 `auto`：元素内容的 width，以及左、右边距(margin)。其余属性必须设定为特定的值，或者默认宽度为 0。

* 0 个 auto：

  当 width 和两边的外边距都有设定值的时候，此时总会把 margin-right 强制为 auto。当 width 有设定值而两边的外边距缺省时，也是如此。

* 1 个 auto：

  当 width 和两边的外边距其中一个为 auto，另外两个有设定值的时候，那么设定为 auto 值的属性会自动拉伸宽度。

* 2 个 auto：

  1) 当 width 有设定值，而两个外边距都为 auto 时，它们会被设置成相等值，从而使该元素在父元素中居中。

  2) 当一个外边距有设定值，另一个外边距和 width 设定为 auto 时，设置为 auto 的外边距会减为 0。

* 3 个 auto：

  当 width 和两个外边距都设定为 auto 时，两个外边距都会设置为 0，而 width 会尽可能宽。这种结果与默认情况是相同的，即没有为外边距或 width 显式声明值，在这种情况下，外边距默认为 0，width 默认为 auto。
