草率地编写正则表达式可能是造成性能瓶颈的主要原因之一，但也有很多提高正则表达式运行效率的方法。两个正则表达式匹配相同的文本并不意味着他们有着同样的速度。
#### 1. 具体化：尽可能具体化分隔符之间的字符串模式  

> 例如，采用模式/“.*?”/来匹配一个双引号保卫的字符串，可以通过吧这个过于宽泛的.\*? > 替换为更加具体 [^"\r\n]\* ，这就除去了回溯时可能发生的几种情况  
#### 2. 嵌套两次与回溯失控  
> 要额外小心嵌套量词的使用，以确保不会应发潜在的回溯失控。所谓嵌套量词是指量词出现在一个自身被重复量词修饰的分组中，例如/(z+)*/;    
#### 3. 正则表达式以简单、必须的字元开始  
> 最理想的情况是，一个正则表达式的起始标记应当尽可能快速地测试并排除明显不匹配的位置。这样来说，好的起始标记通常是一个锚(^)、特定的字符串(x或\u263A)、字符类(比如：[a-z]或类似\d的速记符)和单词边界(\b)，因为他强迫正则表达式识别多重起始字元开头。  
#### 4. 减少分支数量、减少分支范围  
> 分支使用管道符号(|)，可能要求在字符串的每一个位置上测试所有分支选项。通常你可以通过使用字符集和选项组件来减少对分支的需求，获奖分支在正则表达式上的位置退后。

替换前 | 替换后
---|---
cat\|bat | [cb]at
red\|read | rea?d
red\|raw | r(?:en\|aw)
(.\|\r\|\n) | [\s\S]
> 字符集比分支更快，因为它使用位向量(或其他快速实现方式)，而不是回溯。当分支必不可少时，如果不以你选哪个正则表达式匹配的话，时常将分支放在正则表达式的最前面。
#### 5. 使用非捕获分组  
> 捕获分组消耗时间和内存来记录反向引用，并使它保持最新。如果你不需要一个反向引用，可使用非捕获分组来避免这些开箱，比如使用/(?:abc)/来代替/(abc)/,当需要全文匹配的反向引用时，可以才用match或exec方法返回数组的第一项来代替将整个正则表达式包装在一个捕获分组中。  
#### 6. 使用合适的量词  
>即使是处理相同的字符串，贪婪量词和惰性量词的匹配过程有较大区别，使用合适的量词类型可以显著提升性能，尤其是在处理长字符串。  
#### 7. 把正则表达式赋值给变量并重用他们  
>将正则表达式赋值给变量可以避免对它们的重新编译，尤其是在循环体中。  
#### 8. 将复杂的正则表达式拆分为简单的片段(化繁为简)   
>避免在一个正则表达式中处理太多任务。复杂的搜索问题需要条件逻辑，拆分为两个或多个正则表达式更容易理解，通常也会更高效，每个正则表达式只在最后的匹配结果中执行检查，而且更容易维护。
