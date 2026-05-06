>为了博客自定义设计，来系统学习下相关的HTML+css+js语法  
>只是一个简单的语法了解  
- - -
[toc]
# HTML语法学习
- - -
html5(HyperText Markup Language/超文本标记语言)  
__网页的[结构]和[内容]__ (网页的骨架)   
后缀`html`,`htm`  
![img](https://www.runoob.com/wp-content/uploads/2013/06/02A7DD95-22B4-4FB9-B994-DDB5393F7F03.jpg)

## 基础
```html
<h1>这是一个标题</h1>
<p>这是一个段落</p>
<a href="https://www.cnblogs.com/shenleblog/">这是一个链接</a>
<img src="https://www.runoob.com/wp-content/uploads/2013/06/02A7DD95-22B4-4FB9-B994-DDB5393F7F03.jpg" width="300" />

```

## 元素
HTML 标签对大小写不敏感：<P> 等同于 <p>。  
```html
<p>这是第一个段落,p元素定义</p>

<body>
<p>这是第一个段落，body元素定义html文档主体</p>
</body>

<html>
<body>
<p>这是第一个段落，html元素定义html文档</p>
</body>
</html>

<br> 就是没有关闭标签的空元素（<br> 标签定义换行）。
```

## 属性
双引号，单引号都可
```html
<a href='https://www.runoob.com'>链接</a>
<div id="header">标识符</div>
<p class="text highlight">类名</p>
<p style="color: blue; font-size: 14px;">css样式</p>
<abbr title="HyperText Markup Language">提示信息</abbr>   
<div data-user-id="12345">存储自定义数据</div>
```
## 标题
```html
<h1>这是一个标题。</h1>
<h2>这是一个标题。</h2>
<h3>这是一个标题。</h3>
<h4>这是一个标题。</h4>
<h5>这是一个标题。</h5>
<h6>这是一个标题。</h6>
```
`<hr>`等于md的`- - -`,分割水平线  
## 文本格式化
```html
<b>加粗文本</b><br><br>
<i>斜体文本</i><br><br>
<code>电脑自动输出</code><br><br>
这是 <sub> 下标</sub> 和 <sup> 上标</sup>
```
## 链接
```html
<a href="/index.html">本文本</a> 是一个指向本网站中的一个页面的链接。</p>
<p><a href="https://www.microsoft.com/">本文本</a> 是一个指向万维网上的页面的链接。</p>
<a href="https://www.example.com" target="_blank" rel="noopener">新窗口打开 Example</a>
```
_blank: 在新窗口或新标签页中打开链接。  
_self: 在当前窗口或标签页中打开链接（默认）。  
_parent: 在父框架中打开链接。  
_top: 在整个窗口中打开链接，取消任何框架。  
## 头部
`<head>` 元素包含了所有的头部标签元素。在 `<head>`元素中你可以插入脚本（scripts）, 样式文件（CSS），及各种meta信息。

可以添加在头部区域的元素标签为: `<title>`, `<style>`, `<meta>`, `<link>`, `<script>`, `<noscript>`, `<base>`。

## css
1. 内联样式，使用style属性
```html
<p style="color:blue;margin-left:20px;">这是一个段落。</p>
```
2. 内部样式表，在head元素使用style标签
```html
<head>
<style>
body {
    background-color: linen;
}
h1 {
    color: maroon;
    margin-left: 40px;
}
</style>
</head>
```
3. 外部样式表，使用link标签引入外部css文件
```html
<head>
<link rel="stylesheet" href="styles.css">
</head>
```

## 脚本
`<script>` 标签用于定义客户端脚本，比如 JavaScript。    
`<script>` 元素既可包含脚本语句，也可通过 src 属性指向外部脚本文件。  
`<noscript>` 标签提供无法使用脚本时的替代内容，比方在浏览器禁用脚本时，或浏览器不支持客户端脚本时。  
```html
<script>
document.write("Hello World!")
</script>
<noscript>抱歉，你的浏览器不支持 JavaScript!</noscript>
```



# CSS学习
>先学一些基础点，大头之后查找相关文档即可   
- - -
## CSS语法
CSS 规则由两个主要的部分构成：选择器，以及一条或多条声明:  
![img](https://www.runoob.com/wp-content/uploads/2013/07/632877C9-2462-41D6-BD0E-F7317E4C42AC.jpg)  
CSS声明总是以分号 ; 结束，声明总以大括号 {} 括起来:  
```css
p
{
    color:red;
    text-align:center;
    /*这是一个注释*/
}
```

## id和class
### id 选择器
id 选择器可以为标有特定 id 的 HTML 元素指定特定的样式。 
CSS 中 id 选择器以 "#" 来定义。    
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
<style>
#para1
{
	text-align:center;
	color:red;
} 
</style>
</head>

<body>
<p id="para1">Hello World!</p>
<p>这个段落不受该样式的影响。</p>
</body>
</html>
```
### class 选择器
class 属性表示, 在 CSS 中，类选择器以一个点 . 号显示：  
```css
.center {text-align:center;}/*所有拥有 center 类的 HTML 元素均为居中。eg：<h1 class="center">标题居中</h1>*/

p.center {text-align:center;}/*所有的 p 元素使用 class="center" 让该元素的文本居中:*/
```
- - -

## css创建
1. 外部样式表(External style sheet)
2. 内部样式表(Internal style sheet)
3. 内联样式(Inline style)
- - -
（内联样式）Inline style > （内部样式）Internal style sheet >（外部样式）External style sheet > 浏览器默认样式
- - -
### 外部样式表
改变一个文件来改变整个站点的外观。每个页面使用 `<link>` 标签链接到样式表。 `<link>` 标签在（文档的）头部  
```html
<head>
<link rel="stylesheet" type="text/css" href="mystyle.css">
</head>
```
### 内部样式表
使用 `<style>` 标签在文档头部定义内部样式表  
```html
<head>
<style>
hr {color:sienna;}
p {margin-left:20px;}
body {background-image:url("images/back40.gif");}
</style>
</head>
```
### 内联样式
使用 `style` 属性为单个元素定义样式。
```html
<p style="color:blue;margin-left:20px;">这是一个段落。</p>
```


## 链接
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
<style>
a:link {color:#000000;text-decoration:none;background-color:#B2FF99;}      /* 未访问链接*/
a:visited {color:#00FF00;text-decoration:none;background-color:#FFFF85;}  /* 已访问链接 */
a:hover {color:#FF00FF;text-decoration:underline;background-color:#FF704D;}  /* 鼠标移动到链接上 */
a:active {color:#0000FF;text-decoration:underline;background-color:#FF704D;}  /* 鼠标点击时 */
</style>
</head>
<body>
<p><b><a href="/css/" target="_blank">这是一个链接</a></b></p>
<p><b>注意：</b> a:hover 必须在 a:link 和 a:visited 之后，需要严格按顺序才能看到效果。</p>
<p><b>注意：</b> a:active 必须在 a:hover 之后。</p>
</body>
</html>
```

## 边框
```html
<!DOCTYPE html>
<html>
<head>
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
<style>
p.none {border-style:none;}
p.dotted {border-style:dotted;}
p.dashed {border-style:dashed;}
p.solid {border-style:solid;}
p.double {border-style:double;}
p.groove {border-style:groove;}
p.ridge {border-style:ridge;}
p.inset {border-style:inset;}
p.outset {border-style:outset;}
p.hidden {border-style:hidden;}
p.mix {border-style: dotted dashed solid double;}
</style>
</head>

<body>
<p class="none">无边框。</p>
<p class="dotted">虚线边框。</p>
<p class="dashed">虚线边框。</p>
<p class="solid">实线边框。</p>
<p class="double">双边框。</p>
<p class="groove"> 凹槽边框。</p>
<p class="ridge">垄状边框。</p>
<p class="inset">嵌入边框。</p>
<p class="outset">外凸边框。</p>
<p class="hidden">隐藏边框。</p>
<p class="mix">混合边框</p>
</body>

</html>
```

## 定位




# js学习

## 输出
使用 window.alert() 弹出警告框。
使用 document.write() 方法将内容写到 HTML 文档中。
使用 innerHTML 写入到 HTML 元素。
使用 console.log() 写入到浏览器的控制台。
```html
<!DOCTYPE html>
<html>
<head> 
<meta charset="utf-8"> 
<title>菜鸟教程(runoob.com)</title> 
</head>
<body>
	
<h1>我的第一个 Web 页面</h1>
<p id="demo">我的第一个段落。</p>
<script>
window.alert(5 + 6);
</script>
<script>
document.getElementById("demo").innerHTML="段落已修改。";
</script>
<script>
document.write(Date());
</script>
<button onclick="myFunction()">点我</button>
<script>
function myFunction() {
    document.write(Date());
}
</script>
<script>
a = 5;
b = 6;
c = a + b;
console.log(c);
</script>
	
</body>
</html>
```
## 简单语法
| 语句 | 描述 |
|------|------|
| break | 用于跳出循环。 |
| catch | 语句块，在 try 语句块执行出错时执行 catch 语句块。 |
| continue | 跳过循环中的一个迭代。 |
| do ... while | 执行一个语句块，在条件语句为 true 时继续执行该语句块。 |
| for | 在条件语句为 true 时，可以将代码块执行指定的次数。 |
| for ... in | 用于遍历数组或者对象的属性（对数组或者对象的属性进行循环操作）。 |
| function | 定义一个函数 |
| if ... else | 用于基于不同的条件来执行不同的动作。 |
| return | 返回结果，并退出函数 |
| switch | 用于基于不同的条件来执行不同的动作。 |
| throw | 抛出（生成）错误 。 |
| try | 实现错误处理，与 catch 一同使用。 |
| var | 声明一个变量。 |
| while | 当条件语句为 true 时，执行语句块。 |

## 变量
- var：ES5 引入的变量声明方式，具有函数作用域。
- let：ES6 引入的变量声明方式，具有块级作用域。
- const：ES6 引入的常量声明方式，具有块级作用域，且值不可变。

## 数据类型
- 值类型(基本类型)：字符串（String）、数字(Number)、布尔(Boolean)、空（Null）、未定义（Undefined）、Symbol。
- 引用数据类型（对象类型）：对象(Object)、数组(Array)、函数(Function)，还有两个特殊的对象：正则（RegExp）和日期（Date）。

## 对象
```js
let person = {
    firstName:"John",
    lastName:"Doe",
    age:50,
    eyeColor:"blue"
};
```
### 访问对象
- `person.lastName;`,
- `person["lastName"];
`.  
- - -
对象方法是一个函数定义,并作为一个属性值存储。  
```js
var person = {
    firstName: "John",
    lastName : "Doe",
    id : 5566,
    fullName : function() 
	{
       return this.firstName + " " + this.lastName;
    }
};
document.getElementById("demo").innerHTML = person.fullName();
```  


## 函数
```html
<p>点击这个按钮，来调用带参数的函数。</p>
<button onclick="myFunction('Harry Potter','Wizard')">点击这里</button>
<script>
function myFunction(name,job){
	alert("Welcome " + name + ", the " + job);
}
</script>
```
```html
<!--带返回值 -->
<p id="demo"></p>
<script>
function myFunction(a,b){
	return a*b;
}
document.getElementById("demo").innerHTML=myFunction(4,3);
</script>
```

## 事件
HTML 事件是发生在 HTML 元素上的事情。  
当在 HTML 页面中使用 JavaScript 时， JavaScript 可以触发这些事件。  
- `<some-HTML-element some-event='JavaScript 代码'>`
- `<some-HTML-element some-event="JavaScript 代码">`  
- - -
```html
<!--修改 id="demo" 元素的内容 -->
<button onclick="getElementById('demo').innerHTML=Date()">现在的时间是?</button>
<!--修改自身元素的内容 -->
 <button onclick="this.innerHTML=Date()">现在的时间是?</button>
 ```
 | 事件 | 描述 |
|------|------|
| onchange | HTML 元素改变 |
| onclick | 用户点击 HTML 元素 |
| onmouseover | 鼠标指针移动到指定的元素上时发生 |
| onmouseout | 用户从一个 HTML 元素上移开鼠标时发生 |
| onkeydown | 用户按下键盘按键 |
| onload | 浏览器已完成页面的加载 |

## 语法
### ifelse
```js
if (time<10)
{
    document.write("<b>早上好</b>");
}
else if (time>=10 && time<20)
{
    document.write("<b>今天好</b>");
}
else
{
    document.write("<b>晚上好!</b>");
}
```
### switch
```js
var d=new Date().getDay();
switch (d)
{
    case 6:x="今天是星期六";
    break;
    case 0:x="今天是星期日";
    break;
    default:
    x="期待周末";
}
document.getElementById("demo").innerHTML=x;
```
### for
```js
for (var i=0; i<5; i++)
{
      x=x + "该数字为 " + i + "<br>";
}
```
```js
//循环遍历对象 "person" 的属性
var person={fname:"Bill",lname:"Gates",age:56}; 
 
for (x in person)  // x 为属性名
{
    txt=txt + person[x];
}
``` 
### while
```js
while (i<5)
{
    x=x + "The number is " + i + "<br>";
    i++;
}
```
```js
do
{
    x=x + "The number is " + i + "<br>";
    i++;
}
while (i<5);
```
### break、continue
```js
for (i=0;i<10;i++)
{
    if (i==3) break;
    x=x + "The number is " + i + "<br>";
}
```
```js
for (i=0;i<=10;i++)
{
    if (i==3) continue;
    x=x + "The number is " + i + "<br>";
}
```
### typeof
typeof 操作符来检测变量的数据类型。  
| 表达式 | 返回值 | 说明 |
|-------|-------|------|
| typeof undefined | "undefined" | 未定义的值 |
| typeof true | "boolean" | 布尔值 |
| typeof 42 | "number" | 所有数字类型 |
| typeof "text" | "string" | 字符串 |
| typeof {a:1} | "object" | 对象、数组、null |
| typeof function(){} | "function" | 函数 |
| typeof Symbol() | "symbol" | ES6新增符号类型 |
| typeof BigInt(10) | "bigint" | ES2020新增大整数类型 |
## 错误
1. try 语句测试代码块的错误。
2. catch 语句处理错误。
3. throw 语句创建自定义错误。
4. finally 语句在 try 和 catch 语句之后，无论是否有触发异常，该语句都会执行。
```js
//finally 语句不论之前的 try 和 catch 中是否产生异常都会执行该代码块。
//throw 语句允许我们创建自定义错误。
function myFunction() {
  var message, x;
  message = document.getElementById("p01");
  message.innerHTML = "";
  x = document.getElementById("demo").value;
  try { 
    if(x == "") throw "值是空的";
    if(isNaN(x)) throw "值不是一个数字";
    x = Number(x);
    if(x > 10) throw "太大";
    if(x < 5) throw "太小";
  }
  catch(err) {
    message.innerHTML = "错误: " + err + ".";
  }
  finally {
    document.getElementById("demo").value = "";
  }
}
```