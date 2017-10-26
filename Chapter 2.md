# High-Performance-JavaScript-Note
**说明**：该笔记为《高性能JavaScript》读书笔记——高性能JavaScript之XXXX系列，其中包含了我阅读该书时认为比较重要的部分，为了防止自己以后忘记的时候能够及时找到，同时也能与各位g友共享，如有错误，请不吝赐教。
****

*\"**两个两户独立的功能只要通过接口连接就会产生消耗；有个贴切的比喻：把DOM和ECMAScript想象成两个岛屿，之间通过收费桥梁连接，ECMAScript每次访问DOM就得经过这座桥并交纳过桥费；访问的次数越多，交纳的费用越高；因此推荐的做法就是尽量减少访问的次数，并努力待在ECMAScript岛上。**\"*
#### 1 DOM访问与修改
> 访问DOM元素时有代价的，即\"过桥费\"，修改DOM元素的代价更加昂贵，因为会导致浏览器的重排(reflow)与重绘(repaint);最坏的情况是在循环中访问或修改DOM元素，例如：  

    function innerHTMLLoop() {
        for(var i = 0; i < 5000; i++) {
            document.getElementById("test").innerHTML += "a";
        }
    }
> 这段代码的问题在于，每次循环迭代，该元素都被访问两次：一次是读取innerHTML属性，一次是重写innerHTML属性；换一种更高效的办法，用局部变量储存修改的内容，在循环结束后一次性写入：  

    function innerHTMLLoop2() {
        var content = "";
            for(var i = 0; i < 5000; i++) {
            content += "a";
        }
        document.getElementById("test").innerHTML += content;
    }
> **通用的经验法则是：减少DOM访问次数，把运算尽量留在ECMAScript这一段**  
#### 2 克隆节点
> 使用DOM方法更新页面内容的方法一——克隆已有的元素，而不是创建新元素，即使用element.cloneNode()(element表示已有的节点)代替document.createElement()；  
#### 3 访问集合是使用局部变量
> 一般来说，对于任何类型的DOM，需要多次访问同一属性或者方法时，最好用一个局部变量先缓存被访问的目标；当遍历一个HTML集合时，第一优化原则是把集合储存在局部变量中，并把length属性缓存在循环外部，然后使用局部变量代替这些需要多次访问的元素；例如：   


    //较慢
    function collectionGlobal(elementName) {
        var divs = document.getElementsByTagName(elementName),
            length = divs.length,
            nodeName = "",
            nodeType = "",
            tagName = "";
        for(var i = 0; i < length; i++) {
            nodeName = document.getElementsByTagName("div")[i].nodeName;
            nodeType = document.getElementsByTagName("div")[i].nodeType;
            tagName = document.getElementsByTagName("div")[i].tagName;
        }
        return {
            nodeName: nodeName,
            nodeType: nodeType,
            tagName: tagName
        }
    }
> 现在我们把在循环中多次访问的document.getElementsByTagName("div")替换成局部变量divs:  


    //较快
    function collectionGlobal(elementName) {
        var divs = document.getElementsByTagName(elementName),
             length = divs.length,
            nodeName = "",
             nodeType = "",
             tagName = "";
        for(var i = 0; i < length; i++) {
         nodeName = divs[i].nodeName;
            nodeType = divs[i].nodeType;
            tagName = divs[i].tagName;
        }
        return {
            nodeName: nodeName,
            nodeType: nodeType,
            tagName: tagName
        }
    }
> 改进后的程序在循环中虽然不用多次访问document.getElementsByTagName("div")这个全局对象与方法，但是在循环中还是存在多次访问divs[i]，因此，我们对divs[i]进一步缓存，以减少访问divs[i]的次数；  


    //最快
    function collectionGlobal(elementName) {
        var divs = document.getElementsByTagName(elementName),
          length = divs.length,
          nodeName = "",
          nodeType = "",
          tagName = "",
          item = null;
        for(var i = 0; i < length; i++) {
          item =  divs[i];
          nodeName = item.nodeName;
          nodeType = item.nodeType;
          tagName = item.tagName;
        }
        return {
           nodeName: nodeName,
           nodeType: nodeType,
           tagName: tagName
        }
    }
> 最终版本不仅在性能上得到改善，而且代码看起来更加简洁明了；
#### 4 重排和重绘
##### 4.1 重排和重绘何时发生
> 当DOM的变化影响了某个元素的几何属性和位置(例如width、height、position等)时，浏览器要重新计算该元素的属性，同时，其他元素的几何属性和位置也可能受影响，浏览器使收到影响的部分失效，并重新构建渲染树，这个过程叫重排(reflow)；重排结束后，浏览器会重新把受到影响的部分重新渲染到屏幕中，这个过程叫重绘(repaint)；当页面的布局和元素的几何属性发生改变时就需要重排：  
> + 添加或删除可见的DOM元素；
> + 元素的位置改变；
> + 元素的几何属性改变(margin、padding、border-width、width、height等属性改变)
> + 内容改变，例如文本或图片被另外一个不同尺寸的图片代替；
> + 页面渲染器初始化；
> + 浏览器尺寸窗口改变；
重排和重绘的代价是昂贵的，他们会导致web应用程序的UI反应迟钝，因此，应该尽可能减少这类过程的发生；  

##### 4.2 最小化重排和重绘
> **改变元素样式**：将需要改变多个CSS样式的操作合拼，例如：  


    var div = document.getElementById("test");
    div.style.border = "1px solid red;";
    div.style.padding = "5px;";
    div.style.marginTop = "5px;";
> 对于较低版本的浏览器，将导致三次重排和重绘，对此，我们对其进行改进：  


    var div = document.getElementById("test");
    div.style.cssText = "border: 1px solid red; padding: 5px; margin-top: 5px;";
> **批量修改DOM**：当你需要对DOM进行一系列操作时，可以通过以下步骤来减少重排和重绘的次数；
> + 第一步：使其脱离文档流;
> + 第二步：对其进行多重应用；
> + 第三步：把元素带回文档中；  
>
>  该过程会触发两次重排和重绘，如果忽略第一步和第三步，那么在第二步中的任何操作都可能导致重排和重绘；  
>
> 有三种方法可以实现DOM脱离文档流：
> + 隐藏元素；
> + 使用文档断片，在DOM之外构建一个子树，然后再把它拷贝回文档；
> + 将原始元素拷贝到一个脱离文档流的节点中，修改副本，然后在用副本替换原始元素；  

> **缓存布局信息**：当你查询布局信息时，例如偏移量(offsets)、滚动位置(scroll values)或计算出的样式值(computedStyle values)时，浏览器为了返回最新值，会重新刷新列队并应用所有更新；最好的做法是尽量减少获取布局信息的次数，将布局信息保存在局部变量中，然后再操作局部变量；例如：将一个大小为100×100的div沿着对角线从(0,0)的开始位置移动到(500,500)的结束位置：


    //低效的方法
    function moveDiv() {
      var div = document.getElementById("test");
      setTimeout(function() {
        if(div++ < 500 ) {
          div.style.left = 1 + div.style.left + "px";
          div.style.top = 1 + div.style.right + "px";
          console.log(div);
          setTimeout(arguments.callee, 100);
        }else {
          return;
        }
      },10)
    }
    
    
    //改进的方法
    function moveDiv() {
      var div = document.getElementById("test")，
          current = div.offsetLeft;
      setTimeout(function() {
        if(current++ < 500 ) {
          div.style.left = current + "px";
          div.style.top = current + "px";
          setTimeout(arguments.callee, 100);
        }else {
          return;
        }
      },10)
    }  

#### 5 小结
> 1. 最小化DOM访问次数，尽可能在JavaScript端进行处理；
> 2. 如果需要多次访问某个DOM节点，用局部变量储存其引用；
> 3. 小心处理HTML集合，因为他实时连接着底层文档；
> 4. 要留意重排和重绘，减少访问布局信息的次数；
    

