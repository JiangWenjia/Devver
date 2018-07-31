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

- `baseline`

没实验，用的阮一峰老师的图
![](media/15329487318810.jpg)


-`stretch`
item 不设置高度会撑满

![](media/15329486782278.jpg)

