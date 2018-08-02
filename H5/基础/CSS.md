# 选择器

- `Id` 

   `#`+`id`
    ID属性不能以数字开头，Firefox等不起作用
    每个HTML文档中，ID只出现一次

- `Class`
    
   `.` + '类名'
    类名不以数字开头，描述的是一组元素的样式
    
# 创建
  
  创建的三种方式
  - 外部样式表 
  `<head> <link rel="stylesheet" type="text/css" href="mystyle.css"> </head>`
  
  - 内部样式表
    `<style></style>`
    
  - 内联样式
    `<p style="color:sienna;margin-left:20px">这是一个段落。</p>`
    
    优先级 内联样式 > 内部样式表 > 外部样式表> 浏览器缺省
    
# 多重样式
  
  优先级：
1.   通用选择器
2.   元素（标签）选择器
3.   类选择器
4.   属性选择器
5.   伪类
6.   ID选择器
7.   内联样式
8.   ！important

权重
内联：1000
id选择器： 100
类选择器： 10
元素选择器： 1

  
    

    


  
    