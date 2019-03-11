---
layout: post
title:  "jquery选择器"
date:   2019-3-11 22:11:28 +0800
categories: jquery
---
### jquery选择器
#### 1.普通选择器
##### (1).id选择器
$(#id);
>id是唯一的，每个id值在一个页面中只能使用一次。如果多个元素分配了相同的id，将只匹配该id选择集合的第一个DOM元素。但这种行为不应该发生;有超过一个元素的页面使用相同的id是无效的

##### (2).类选择器
$( ".class" )
>类选择器，相对id选择器来说，效率相对会低一点，但是优势就是可以多选

##### (3).元素选择器
$( "element" ) 如 $( "div" ) 
>搜索指定元素标签名的所有节点

##### (4).全选择器
$( "*" )
>获取文档中所有的元素

#### 2.层级选择器
##### (1).子选择器
$('parent > child')
>选择所有指定parent元素中指定的child**直接**子元素,不能递归子代。

如：$('div > span')可以选择到直接子元素span2，但是选不到非直接子元素span1
```
<div>
    <p>
        <span>span1</span>
    </p>
    <span>span2</span>
</div>
```
##### (2).后代选择器
$('ancestor descendant')
>选择指定祖先元素的所有后代元素。会递归到子代中继续选择。

如上面的html，使用$('div span')就会同时选择到span1和span2。
##### (3).相邻兄弟选择器
$('prev + next')
>所有**紧接**prev后的next元素，不能向前选择，prev与next之间也不能有其他元素，同时不能递归子元素

如：$('prev + span')可以选择到类为prev的div元素的兄弟元素span1,但是选择不到span2，以为prev和span2中间隔了一个h2。
```
<div>
    <div class="prev"></div>
    <span>span1</span>
    <div class="prev"></div>
    <h2> 间隔 </h2>
    <span>span2</span>
</div>
```
##### (4).一般兄弟选择器
$('prev ~ siblings')
>选择prev后的所有siblings兄弟元素，不能向前，也不能递归，因为是兄弟，需要是同一个父元素。

如$('prev ~ span')就可以同时选择上面的span1和span2了。

#### 3.筛选选择器
##### (1).基本筛选选择器
\$(':first') : 匹配第一个元素  
\$(':last') : 匹配最后一个元素  
\$(':not(selector)') : 在匹配的集合中，去除这里的selector  
\$(':eq(index)') : 在匹配的集合中，选择索引值为index的元素，即第几个  
\$(':gt(index)') : 在匹配的集合中，选择索引值**大于**index的元素  
\$(':lt(index)') : 在匹配的集合中，选择索引值**小于**index的元素  
\$(':even') : 在匹配的集合中，选择索引值为**偶数**的元素，从0开始计数  
\$(':odd') : 在匹配的集合中，选择索引值为**奇数**的元素，从0开始计数  
\$(':header') : 选择所有标题元素，如h1,h2,h3等  
\$(':lang(language)') : 选择指定语言的所有元素  
\$(':root') : 选择该文档的根元素  
\$(':animated') : 选择所有正在执行动画效果的元素

##### (2).内容筛选选择器
\$(':contains(text)') : 选择所有包含指定文本的元素  
\$(':parent') : 选择所有包含子元素或文本的元素  
\$(':empty') : 选择所有没有子元素和文本的元素  
\$(':has(selector)') : 选择元素中至少包含指定选择器的元素

##### (3).可见性筛选选择器
\$(':visible') : 选择所有显示的元素  
\$(':hidden') : 选择所有隐藏的元素
>:hidden选择器，不仅仅包含样式是display="none"的元素，还包括隐藏表单、visibility等等

我们有几种方式可以隐藏一个元素：
1. CSS display的值是none。
2. type="hidden"的表单元素。
3. 宽度和高度都显式设置为0。
4. 一个祖先元素是隐藏的，该元素是不会在页面上显示
5. CSS visibility的值是hidden
6. CSS opacity的指是0

>如果元素中占据文档中一定的空间,元素被认为是可见的。
可见元素的宽度或高度，是大于零。
元素的visibility: hidden 或 opacity: 0被认为是可见的，因为他们仍然占用空间布局。
