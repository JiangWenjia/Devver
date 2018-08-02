# 资料

[阮一峰flex参考资料](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

# flex-direction
**元素排列的方向**（主轴的方向）

水平排列
- `row`
从左往右，水平
![](media/15329468703426.jpg)

- `row-reverse`

![](media/15329469424871.jpg)

垂直排列

- `column`

![](media/15329470347046.jpg)

- `column-reverse`


![](media/15329470871753.jpg)

# flex-wrap 
是否换行
- `flex-wrap: nowrap;`

默认 `flex-wrap: nowrap;` 长度不够的时候，会按内容给予宽度，以下实际设置了宽度为100的，但是由于宽度不够，平分了父元素的宽度

![](media/15329472398183.jpg)

- `flex-wrap: wrap;`

![](media/15329474426935.jpg)

- `wrap-reverse`

![](media/15329474821256.jpg)


# justify-content

主轴对齐方式（单轴线属性）

- `flex-start`

![](media/15329478587601.jpg)

- `flex-end`

![](media/15329478832416.jpg)

- `center`

![](media/15329479636587.jpg)

- `space-between`
主轴两端对齐，间距相等
![](media/15331791969455.jpg)

- `space-around`
> 元素之间的间距相等

![](media/15331785820794.jpg)

# align-items

- `flex-start`

交叉轴起点对齐
![](media/15331809175709.jpg)


- `flex-end`

![](media/15331810048085.jpg)


- `flex-center`

![](media/15331873286795.jpg)


- `baseline`

![](media/15331875037077.jpg)


- `stretch`

![](media/15331873780886.jpg)





# align-conent
 
 把轴线当成元素，然后在交叉轴为参考线
 
 ![](media/15331814901584.jpg)

# 项目属性

##  order
> 定义项目的排列顺序，数值越小，排列越靠前，默认是0

![](media/15331877528766.jpg)

## flex-grow 
> 属性占剩余空间的放大比例 （父元素在主轴上太长了） 默认是0 不放大

![](media/15331878968332.jpg)

## flex-shrink
> 属性的缩小比例 （父元素在主轴上太小了） 默认是1 

![](media/15331881275738.jpg)

上图原本项目宽度是200px

## flex-basis
> 在定义分配空间之前，项目占据的主轴空间（main-size） 默认是auto，可以是数值，或者百分比

![](media/15331883567737.jpg)

注意相当于定义了主轴上的width，优先级比flex-grow高

# flex
 `flex-grow` `flex-shrink`? || `flex-basis` 的

两个默认值 auto（1 1 auto）
none （0 0 auto）

![](media/15331885868953.jpg)

# align-self
> 单个项目覆盖align-items 属性 ，默认是
> auto 相当继承父元素的align-items，如果没有父元素等同于stretch


```css
.item {
  align-self: auto | flex-start | flex-end | center | baseline | stretch;
}
```

![](media/15331887681129.jpg)

