# 用法

不在需要写 `type="text/javascript"`，因为已经默认
外部引入的不能添加`<script>`

```js
    <script>
        alert("Hello Wrold!");
    </script>
    <script src="test.js"></script>
```


# 输出

- `window.alert()` 既`alert`
-  `document.write()` 将内容写到HTML文档中
-  `innerHTML`     赋值HTML元素标签内容
-  `console.log()`  控制台


```js
document.write("<div id='demo'> </div>")
document.getElementById('demo').innerHTML = "demo-innerHtml"
```

# 变量

以字母 $_ 开头
```objc
     var i = 1, B = "2"; // 区分大小写
     var _i = 3; // 可以，但是不建议
     var $4 = 4; // 可以，但是不建议
     var i;     //重新声明
     alert(i); //值仍为1 未定义的为undefined
```

# 数据类型

- 字符串（`String`）
- 数字（`Number`）
- 布尔 (`Boolean`)
- 数组 (`Array`)
- 对象 (`Object`)
- 空   (`Null`)
- 为定义 （`Undefined`）


```js
  //动态类型(弱类型)
         var x;
         var x = 5;
         x = "John";

         var s = "String";
         var num = 3;
         var b = true;

         //数组
         var cars = new Array();
         cars[0] = "1";
         var cars = ["0","a","b"];
         //对象
         var person = {firstName:"Jhon",age:10};
         car = null;
         var u; // undefined 为定义类型，如果直接使用一个没有声明的变量，会报错
         alert(u);//undefined;
```

# 作用域

分局部变量和全局变量

```js
     var  box1 = "box1";//全局变量

     function f2() {
         carName = "cn";//没有使用var关键字是全局变量，可以使用window.carName
         var c = "c";//局部变量
     }

     alert(window.carName);
```

# 事件

```js
 <button id="btn" style="width: 100px;height: 30px;" onclick="alert('可是直接写js');">按钮</button>
<style>
    #btn:hover { //鼠标经过时候的颜色
        background-color: red;
    }

</style>
<script>

    var button = document.getElementById("btn");


    button.onmouseover = function (ev) {
      alert("onmouseover");
    };

    button.onmouseout = function (ev) {
        alert("onmouseout");
    };

    window.onload = function (ev) {
        alert("onload");
    };


</script>
```

# 字符串

```js
    var carname = "var";//String                                                       
    var character = carname[0];//v                                                     
    var length = carname.length;                                                       
    var carObject = new String("var"); //Object 不建议使用会拖慢速度 相同字符串，string 不等于 object类型的  

```

# == 和 ===

- `==` : 表示值相等
- `===`: 值相等，同时类型也相等

## 控制语句


```js
    var array = [1,2,3,4];

    for (num in array) {   // num 加不加var 都可以
        console.log(num);
    }

    
    alert(window.num);
    list:
    {
        console.log('a');
        console.log('b');
        break list;  // break 可以跳出到标签
        console.log('c');
    }

```

# js类型转换

typeof + 变量 返回类型
constructor： String() { [native code]」
constructor返回构造函数

5种不同的数据类型
- string
- number
- boolean
- object
- function

3 种对象类型

- Object (String,Number等)
- Date
- Array

2 种不包含任何值的数据类型

- null （object）
- nudefined 

```js
    {
        console.log(typeof "John"); //string
        console.log(typeof 3.0); //number
        console.log(typeof NaN); //number
        console.log(typeof false);//boolean
        console.log(typeof [1,2,4]); //object
        console.log(typeof {name:"name"});//object
        console.log(typeof new Date());
        console.log(typeof function () {//function

        });
        console.log(typeof null);
        console.log(typeof new Number(1));//object
    } 
```

# 异常

```js
  try{
        // alett(xxx);
        throw {message:"wwww"};
    }catch (e) {
        alert(typeof e + e.message);
    }
```

# json

```js
    //将会进入debug模式
    debugger;
    
    var text = '{"name":"jwj"}';
    var person = JSON.parse(text); //json 转 object
    console.log(person.name);
```

# void(0)

void()表示计算值，0没有任何效果

```js
 <button onclick="javascript:void(alert(2))">xxx</button>
    <button onclick="javascript:void(0)">xxx</button>
```




