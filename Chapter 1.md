# 高性能JavaScript系列之脚本的加载与执行(1)
**说明**：该笔记为《高性能JavaScript》读书笔记——高性能JavaScript之XXXX系列，其中包含了我阅读该书时认为比较重要的部分，为了防止自己以后忘记的时候能够及时找到，同时也能与各位g友共享，如有错误，请不吝赐教。
****

#### 1 脚本位置
> **脚本位置**：由于script脚本在加载过程中会阻塞其他资源的加载，因此推荐将script脚本尽可能放在boby标签的底部，以尽量减少对整个页面其他资源下载的影响；  
>
#### 2 组织脚本
> **组织脚本**：浏览器在解析HTML过程中每遇见一个script标签，都会因为脚本的执行而导致一定的延时；因此最小化延迟时间能提高页面的整体新能；对于外链JavaScript脚本文件，考虑到http请求的开销，下载4个25KB的文件比下载1个100KB的文件更快，因此，减少页面外链接脚本的数量能改善页面的性能；
>
#### 3 无阻塞脚本
> **3.1 脚本**：减少JavaScript文件的大小并限制HTTP请求的次数，只是构建快速响应web应用的第一步；尽管下载较大的耽搁脚本产生一次http请求，却会锁死浏览器一大段时间；为了避免这种情况，逐步向页面中加载JavaScript文件，这样做在某种程度上不会阻塞浏览器；无阻塞脚本的秘诀在于在浏览器加载完毕之后再加载JavaScript文件，采用专业术语来讲，这意味着在window对象的load事件触发后再加载JavaScript文件；  
>
> **3.2 defer与async**:defer和asyn时无阻塞脚本加载方式，相同点是两者都采用并行下载，却别是defer需要等到页面完成后才会执行，而async是加载完成自动执行；  
>
> **3.3 延迟脚本**：HTML4为script标签拓展了一个defer属性，该属性表明标签所含脚本不会修改DOM，带有defer属性的script标签可以放在文档的任何位置；，目前，所有主流浏览器都已支持该属性；注意，该属性只有在src属性被声明时才有效；  

     <script src="./js/demo.js" type="text/javascript" defer>
		//该脚本输出一个alert("defer");
	</script>
	<script >
		alert("script");
	</script>	
	<script >
		window.onload = function() {
			alert("onload");
		}
	</script>	
> 对于不支持defer属性的浏览器，会输出defer、script、onload；对于支持defer属性的浏览器，会弹出script、defer、onload；  

> **3.4 动态脚本**：动态添加的script标签，这种技术的重点在于无论何时启动下载，文件的下载和执行都不会阻塞页面的其他进程；通常来讲，动态创建的script标签调价到head中比添加到body保险，尤其是在页面加载过程中执行代码更是如此；动态加载脚本的标准与IE特有方法封装如下：  

    function loadScript(url,calback) {
      var script = document.crateElement("script");
      script.type = "text/javascript";
      if(script.readyState) {//for IE
        script.onreadyStateChange = function() {
          if(script.readyState == "loaded" || script.readyState == "complete") {
            script.onreadyStateChange = null;
            script.src = url;
          }
          calback();
        }
      }else {
        script.onlaod = function() {
          calback();
        }
        script.src = url;
      }
      document.getElementsByTagName("head")[0].appendChild(script);
    }  
> 如果想动态加载多个JS文件，可以嵌套调用loadScript函数：   

    loadScript("file1.js", function() {
      loadScript("file2.js",function() {
        loadScript("file3.js",function() {
          alert("All file is loaded");
        })
      })
    })
> 如果加载多个JS文件的顺序很重要，更好的办法是按照顺序把他们合并为一个JS文件进行加载；
**动态加载脚本技术凭借它其跨浏览器兼容性和易用优势，已经成为最常用的无阻塞加载解决**
>
> **3.5 XMLHttpRequest脚本注入**：无阻塞加载JS文件的另一种方式是通过XHR对象也可以在页面中动态注入脚本；

    function loadScriptByXHR(url) {
        //创建跨浏览器XHR对象
        function createXHR() {
          if( typeof XMLHttpRequest != "undefined") {
            createXHR = function() {
              return new XMLHttpRequest();
            }
          }else if( typeof ActiveXObject !="undefined") {
            createXHR = function() {
              if (typeof arguments.callee.activeXString != "string") {
                var versions = ["MSXML2.XMLHttp.6.0","MSXML2.XMLHttp.3.0","MSXML2.XMLHttp"],
            len,
            i;
                for(i = 0, len = versions.length; i < len; i++) {
                  try{
                    new ActiveXObject(versions[i]);
                    arguments.callee.activeXString = versions[i];
                    break;
                  }catch(err) {
                    //跳过
                  }
                }
              }
              return ActiveXObject(arguments.callee.activeXString); 
            } 
          }else {
            createXHR = function() {
              throw new Error("No XHR object available.");
            } 
          }
          return createXHR();
        }
        
        //通过XHR对象动态注入脚本
        var xhr = createXHR();
        xhr.open("get", url, true);
        xhr.onreadyStateChange = function() {
            if(xhr.readyState == 4) {
                if(xhr.status >= 200 && xhr.status < 300 || xhr.status == 304) {
                 var script = document.createElement("script");
                    script.type = "text/javascript";
                    script.text = xhr.responseText;
                    document.body.appendChild(script);
                }
            }
        }
        xhr.send(null);
    }  
> **主要优点**：你可以下载JS文件，但不立即执行(由于代码是在<script>标签之外返回的，因此它下载后不会自动执行)，这使得你可以把脚本的执行推迟到你准备好的时候；另一个优点是同样的代码在所有主流浏览器中都能正常运行；  
> **主要缺点**：这种方法的局限是JavaScript文件必须与所请求的页面处于相同的域，这意味着JavaScript文件不能从CDN上下载，因此大型web应用一般不会使用XHR对象动态注入脚本技术；    

> **3.6 推荐的无阻塞模式**：
> 向页面中加载大量JS脚本的推荐做法只需要两步：先添加动态添加所需要的代码，然后加载初始化页面所需要的代码；  
>**方法1**：  

    <script src="loader.js" type="text/javascript" charset="utf-8" ></script>
	<script type="text/javascript" charset="utf-8" >
		loadScript("the-rest.js",function() {
			Application.init();
		})
	</script>
> **方法2**：  

    <script type="text/javascript" charset="utf-8" >
        function loadScript(url,calback) {
        var script = document.crateElement("script");
        script.type = "text/javascript";
        if(script.readyState) {//for IE
            script.onreadyStateChange = function() {
            if(script.readyState == "loaded" || script.readyState == "complete") {
                script.onreadyStateChange = null;
                script.src = url;
            }
            calback();
            }
        }else {
            script.onlaod = function() {
            calback();
            }
         script.src = url;
        }
            document.getElementsByTagName("head")[0].appendChild(script);
        } 
        loadScript("the-rest.js",function() {
			Application.init();
		})
    </script>
> **\*注意\***：如果采用第二种方法，建议把初始化代码压缩到最小；  

> #### 4 小结
> 减少JS脚本加载对性能影响的方法有：
+ 将所有script标签放在body底部，这能确保在脚本执行之前页面已经渲染完毕；
+ 合并脚本。页面中脚本的数量越少，加载脚本的速度越快，响应更加迅速，无论内嵌脚本还是外链JS文件；
+ 有多种无阻塞下载JavaScript的方法  
1.1 使用script标签的defer属性  
1.2 使用动态创建的script元素来下载并执行代码；  
1.3 使用XHR对象下载JS代码并注入页面中；
