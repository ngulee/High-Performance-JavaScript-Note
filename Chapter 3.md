\"_代码的整体结构时影响运行速度的主要原因。代码数量少不并不意味着运行速度快，代码数量多也并不意味着运行速度慢，代码的组织结构和解决具体问题的思路是影响整体运行性能的主要原因。_\""

### 1 循环

循环处理是最常见的编程模式之一，大多数的编程语言中，执行代码的时间大部分消耗在循环中，因此，如何高效实用循环是提升运行性能的关注点之一；

#### 1.1 循环体的类型1111

##### 1.1.1 循环的类型

ECMA-262标准第三版定义了JavaScript语法中的四种基本类型：for、while、do-while以及for-in循环；

1）for循环

```
for (var i = 0; i < Things.length; i++) {
    //循环主体
}
```

2）while循环

```
var i = 0;
while(i < 10) {
    //循环主体
    i++;
}
```

3）do-while循环

```
var i = 0;
do {
    //循环主体
    i++;
} while(i++ < 10);
```

4）for-in

```
for(var property in object) {
    //循环主体
}
```

在以上这四种循环体中，只有for-in循环比其他三种循环明显要慢，因为for-in循环每次迭代操作都会搜索实例和原型中的属性，会产生更多的开销，所以比其他循环要慢；除非你要迭代一个属性数量未知的对象，否则最好不要使用for-in循环；  
在ES6中，规定了一个新的循环语句：for-of

4）for-of

```
for(var property of object) {
    //循环主体
}
```

相比for-in循环，for-of效率高更，而且可以用于数组的遍历；

##### 1.1.2 循环的性能

1）减少迭代的工作量  
一次循环迭代所花时间越长，那么多次迭代就需要更多的时间，所以，限制循环中耗时操作的数量能提高循环整体的速度；  
一个典型的数组迭代可以采用for、while、do-while中的任何一种：

```
//原始版本
for (var i = 0; i < arr.length; i++) {
    //处理函数processArr
    processArr(arr[i]);
}

var j = 0;
while(j < arr.length) {
    processArr(arr[j++]);
}

var k = 0;
do {
    processArr(arr[k++]);
}while(k < arr.length);    
```

在上面的循环体中，每次运行循环体时都会产生如下操作：

* 在控制条件中查找一次属性\(arr.length\);      
* 在控制条件中执行一次数值比较\(i &lt; arr.length\);
* 一次比较操作，查看控制条件的计算结果是否为true\(i &lt; arr.length == true\)
* 一次自增操作\(i++\);
* 一次数组查询\(arr\[i\]\);
* 一次函数调用processArr\(arr\[i\]\);   

在这些循环中，运行的速度很大程度上取决于函数processArr\(\)对每项的操作，即便如此，减少循环中操作的总次数能大幅提升运行的总体性能；  
对于该例，可以从两个方面着手：  
**减少对数组项的查找次数**：上面的循环中，每次迭代都要查找一次arr.length，由于在循环的过程中，该值并不会改变，对此，我们可以把它存在一个局部变量中，然后在控制条件中使用这个局部变量；  
**颠倒数组的顺序**：如果数组的顺序与执行的任务无关，则可以通过颠倒数组的顺序来提升性能；

```
//减少属性并颠倒迭代顺序
for (var i = 0, len = arr.length; i < len) {
    //处理函数processArr
    processArr(arr[i]);
}

var j = arr.length;
while(j--) {
    processArr(arr[j]);
}

var k = arr.length - 1;
do {
    processArr(arr[k]);
}while(k--);    
```

对比原始版本，优化后的循环有如下操作：

* 一次控制条件中的比较\(i == true\);
* 一次减法操作\(i--\);
* 一次数组项查找\(arr\[i\]\);
* 一次函数调用processArr\(arr\[i\]\);   

在新的循环体中，只对数组长度进行了一次查找，控制条件比较已经从两次\(迭代数少于总数吗？它为true吗\)较少到一次\(它为true吗\)；随着迭代次数的增加，性能会得到明显的提升；

### 2 条件语句

与循环原理相似，条件表达式决定JavaScript运行流的走向；在JavaScript中有两种条件语句：if-else和switch，不同浏览器针对流程控制进行了不同优化，因此使用哪种技术和具体业务有关；

#### 2.1 if-else和switch比较

是使用if-else还是switch，最流程的方法是根据测试条件的数量来判断：条件数量少，倾向于使用if-else，反之，倾向于使用switch；这通常归结代码的易读性，当条件数量少时if-else更易读，条件数量多时switch更易读；  
事实证明，switch要比if-else运行得更快，但是只有在条件数量很大时才会明显；这个量语句的主要区别是，当条件数量增加时，if-else性能负担增加的程度比switch更多；  
通常来说，if-else语句适用于两个离散值或几个不同值域的判断，当判断多于两个离散值时，switch语句更加合适；

##### 2.1.1 优化if-else

优化if-else的目标是：最小化达到正确分支前的所需判断条件的数量，最简单的方法是，把最可能出现的条件放在最前面；例如：

```
if(value < 5) {
    //代码处理
}else if(vaule >= 5 && value < 10) {
    //代码处理
}else {
    //代码处理
}
```

这段代码只有value &lt; 5 出现的概率最大才是最优的；if-else语句中的条件判断应该总是按照从概率最大到概率最小的的顺序排列，以保证运行的速度最快；  
 另一个减少条件判断次数的方法是把if-else组织成一系列嵌套的if-else语句。使用单个庞大的if-else需要进行多次条件判断，通常会导致运行缓慢。例如：

```
if(value == 0) {
    return result0;
}else if(value == 1 ) {
    return result1;
}else if(value == 2) {
    return result2;
}else if(value == 3) {
    return result3;
}else if(value == 4) {
    return result4;
}else if(value == 5) {
    return result5;
}else if(value == 6) {
    return result6;
}else if(value == 7) {
    return result7;
}else if(value == 8) {
    return result8;
}else if(value == 9) {
    return result9;
}else {
    return result10;
}   
```

在这个条件表达式中，条件语句最多要进行10次判断，为了最小化条件判断的次数，可重写为一系列嵌套的if-else语句，比如：

```
if(value < 6) {
    if(value < 3) {
        if(value == 0) {
            return result0;
        }else if( value == 1) {
            return result1;
        }else {
            return result2;
        }
    }else {
        if(value == 3) {
            return result3;
        }else if( value == 4) {
            return result4;
        }else {
            return result5;
        }
    }
}else {
    if(value < 8) {
        if(value == 6) {
            return result6;
        }else {
            return result7;
        }
    }else {
        if(value == 8) {
            return result8;
        }else if( value == 9) {
            return result9;
        }else {
            return result10;
        }
    }
}    
```

利用二分法，把值域分成一系列的区间，然后逐步缩小范围，使得重写后的if-else语句达到正确分支前的条件判断次数减少到最多只有4次；该方法适合多个值域的测试；

### 3 小结

* for、while、do-while循环性能特性相当；
* 避免使用for-in循环，除非要遍历一个属性数量位置的属性；
* 改善循环的最佳方式是减少每次迭代的运算量和循环的总次数；
  +当在判断条件多，为离散值时，switch性能优于if-else，当判断条件少时，if-else更具可读性；



