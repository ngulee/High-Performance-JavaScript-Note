## 高性能JavaScript系列之字符串\(4\)

"_几乎所有的JavaScript程序都与字符串操作密切相关。例如，许多应用使用Ajax从服务端获取字符串，并把这些字符串转化为更易用的JavaScript对象，然后由这些数据生成HTML字符串。一些典型的应用程序通常需要处理大量类似合并、分割、重新排序、搜索、比那里等字符串操作。随着web应用越来越复杂，越来越多的类似任务会在浏览器中完成。_"

"_在JavaScript中，正则表达式时必不可少的，它的重要性远远超过哪些琐碎的字符串处理。_"

"_By Steven Levithan_"

### 1 字符串连接

> 当需要连接的字符串数量较少时，可以采用以下方法实现：
>
> * The + operator模式：str = "a" + "b" + "c";
> * The += operator模式：str = "a";str += "b";str += "c;
> * Array.join\(\)方法：str = \["a","b","c"\].join\(""\);
> * String.concat\(\)方法：str = "a";str = str.concat\("b","c"\);

#### 1.1 加\(+\)和加等\(+=\)操作符

> 随着要合并的字符串的长度和数量增加，有一些方法开始展现出优势，首先看一个栗子\(该栗子只能看，不能吃\)，这是一个连接字符串的常用方式：

```js
var str = "";
str +="one" + "two";
```

> 次代码运行时会经历四个步骤：  
> 1. 首先，在内存中穿件一个临时字符串；  
> 2. 连接后的字符串\"onetwo\"被赋值给该临时字符串；  
> 3. 临时字符串与str当前的值连接；  
> 4. 连接后的字符串赋值给str；

以下代码用两两行语句直接附加内容给str，葱而避免了产生临时字符串：

```js
var str = "";
str += "one";
str += "one" + "two";
```

或：

```js
var str= "";
str += "one" + "two";
```

在大多数浏览器中，这样做能够提速10%-40%，但是对于IE7以及更低版本的IE浏览不适用，这是由于IE执行连接操作的底层机制决定的。

#### 1.2 数组项合并

数组项合并字符串方法：

```js
["str1","str2",...,"strn"].join("")
```

在大多数浏览器中，数组项合并比其他字符串连接符更慢，但事实上，它却是IE7级更早版本浏览器中合并大量字符串的唯一高效途径，这也算是一种补偿；

#### 1.3 字符串concat方法

字符串的原生方法concat能接收任意数量的参数，并将每一个参数附加到被调用的字符串上。

```js
//附加一个字符串
str = str.concat(s1);

//附加多个字符串
str = str.concat(s1,s2,...,sn);

//如果传递一个数组，可以附加数组中的所有字符串
str = String.prototype.concat.apply(str,[s1,s2,...,sn]);
```

遗憾的是，在大多数情况下，concat方法要比+和+=操作稍慢。

### 2 正则表达式的优化

草率编写正则表达式是造成性能瓶颈的主要原因，两个正则表达式匹配相同的文本，并不意味着它们具有同样的速度；  
关于正则表达式的优化，等我看完《Regular Expression Cookbook》这本书再来更新；

