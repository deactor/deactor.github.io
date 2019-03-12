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

##### (4).属性筛选选择器
\$("[attribute |= 'value']") : 指定属性值等于value或以value为前缀（该字符串后跟连字符'-'）的元素。  
\$("[attribute *= 'value']") : 指定属性值包含value的元素。  
\$("[attribute ~= 'value']") : 指定属性用空格分隔的值中包含value（全词匹配）的元素。  
\$("[attribute = 'value']") : 指定属性为value的元素。  
\$("[attribute != 'value']") : 指定属性不为value的元素。  
\$("[attribute ^= 'value']") : 指定属性以value开始的元素。  
\$("[attribute $= 'value']") : 指定属性以value结尾的元素，区分大小写。  
\$("[attribute]") : 选择具有指定属性的元素，元素可以为任何值。  
\$("[attributeFilter1][attributeFilterN]") : 选择匹配所有指定的属性筛选器的元素。  

##### (5).元素筛选选择器
\$(":first-child") : 选择所有父级元素下的第一个子元素。  
\$(":last-child") : 选择所有父级元素下的最后一个子元素。  
\$(":only-child") : 如果某个元素是其父元素的唯一子元素，那么它就会被选中。  
\$(":nth-child(n)") : 选择他们的所有父元素的第n个子元素。  
\$(":nth-last-child(n)") : 选择所有他们父元素的第n个子元素。计数从最后一个元素开始到第一个。  

>1.:first只匹配一个单独的元素，但是:first-child选择器可以匹配多个：即为每个父级元素匹配第一个子元素。这相当于:nth-child(1)  
2.:last 只匹配一个单独的元素， :last-child 选择器可以匹配多个元素：即，为每个父级元素匹配最后一个子元素  
3.如果子元素只有一个的话，:first-child与:last-child是同一个  
4.:only-child匹配某个元素是父元素中唯一的子元素，就是说当前子元素是父元素中唯一的元素，则匹配  
5.jQuery实现:nth-child(n)是严格来自CSS规范，所以n值是“索引”，也就是说，从1开始计数，:nth-child(index)从1开始的，而eq(index)是从0开始的  
6.nth-child(n) 与 :nth-last-child(n) 的区别前者是从前往后计算，后者从后往前计算  

```
<body>
    <h2>子元素筛选选择器</h2>
    <h3>:first-child、:last-child、:only-child</h3>
    <div class="left first-div">
        <div class="div">
            <a>:first-child</a> <!--是父级元素下的第一个元素，被选中-->
            <a>第二个元素</a>
            <a>:last-child</a>
        </div>
        <div class="div">
            <a>:first-child</a> <!--是父级元素下的第一个元素，被选中-->
        </div>
        <div class="div">
            <a>:first-child</a> <!--是父级元素下的第一个元素，被选中-->
            <a>第二个元素</a>
            <a>:last-child</a>
        </div>
    </div>

    <script type="text/javascript">
        //查找class="first-div"下的第一个a元素
        //针对所有父级下的第一个
        $('.first-div a:first-child').css("color", "#CD00CD");
    </script>
</body>
```

##### (6).表单元素选择器
\$(":input") : 选择所有input，textarea，select和button元素。  
\$(":text") : 匹配所有文本框。  
\$(":password") : 匹配所有密码框。  
\$(":radio") : 匹配所有单选按钮。  
\$(":checkbox") : 匹配所有复选框。  
\$(":submit") : 匹配所有提交按钮。  
\$(":image") : 匹配所有图像域。  
\$(":reset") : 匹配所有重置按钮。  
\$(":button") : 匹配所有按钮。  
\$(":file") : 匹配所有文件域。  

##### (7).表单对象属性筛选选择器
\$(":enabled") : 选取可用的表单元素。  
\$(":disabled") : 选取不可用的表单元素。  
\$(":checked") : 选取被选中的\<input>元素。  
\$(":selected") : 选取被选中的\<option>元素。  

>1.选择器适用于复选框和单选框，对于下拉框元素, 使用 :selected 选择器  
2.在某些浏览器中，选择器:checked可能会错误选取到<option>元素，所以保险起见换用选择器input:checked，确保只会选取<input>元素

##### (8).特殊选择器this
$(this)和this有什么区别？  
this是JavaScript中的关键字，指的是当前的上下文对象，简单的说就是方法/属性的所有者。
下面例子中，imooc是一个对象，拥有name属性与getName方法,在getName中this指向了所属的对象imooc  
```
var imooc = {
    name:"慕课网",
    getName:function(){
        //this,就是imooc对象
        return this.name;
    }
}
imooc.getName(); //慕课网
```
当然在JavaScript中this是动态的，也就是说这个上下文对象都是可以被动态改变的(可以通过call,apply等方法)  
同样的在DOM中this就是指向了这个html元素对象，因为this就是DOM元素本身的一个引用  

假如给页面一个P元素绑定一个事件:
```
p.addEventListener('click',function(){
    //this === p
    //以下两者的修改都是等价的
    this.style.color = "red";
    p.style.color = "red";
},false);
```
通过addEventListener绑定的事件回调中，this指向的是当前的dom对象，所以再次修改这样对象的样式，只需要通过this获取到引用即可  
```
 this.style.color = "red"
```

但是这样的操作其实还是很不方便的，这里面就要涉及一大堆的样式兼容，如果通过jQuery处理就会简单多了，我们只需要把this加工成jQuery对象,换成jQuery的做法：  
```
$('p').click(function(){
    //把p元素转化成jQuery的对象
    var $this= $(this) 
    $this.css('color','red')
})
```
通过把$()方法传入当前的元素对象的引用this，把这个this加工成jQuery对象，我们就可以用jQuery提供的快捷方法直接处理样式了  

>this，表示当前的上下文对象是一个html对象，可以调用html对象所拥有的属性和方法。  
$(this),代表的上下文对象是一个jquery的上下文对象，可以调用jQuery的方法和属性值。

### 参考：
https://www.imooc.com/code/8353