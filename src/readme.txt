解读jquery源码
1. ( function(){
        (21,94)  定义了一些变量和函数 jQuery =function(){}返回一个jquery对象

        (96,283) 给jQuery对象，添加了一些方法和属性 ，jquery采用面向对象的方式来设计的。
            $(".div").css()与js对象 var arr=new Array();arr.push()很像，$(".div")为一个函数调用，但函数调用的返回结果为对象也就可以使用.的方式来调用了
        
        (285,347) extend:jQuery的继承方法。可以利于后期添加方法的扩展，代码都写在一起不利于后期的维护与扩展
        
        (349,817) jQuery.extend()：扩展一些工具方法。jquery中调用方法的写法有$().css()和$.trim();前者只能给jQuery对象来用，
                后者调用的是静态方法。静态方法可以看做是jquery框架的底层内容

        (877,2856) sizzle:完成jquery中复杂选择器的实现$('ul li+p .list')，专门的sizzle.js就可以实现这个功能

        (2880,3042) Callbacks：jQuery中的回调对象，主要是负责函数的一个统一管理。函数比较多时，管理起来比较麻烦。
            function fn1(){alert(1)};
            function fn2(){alert(2)};
            var cb=$.Callbacks(); //里面有一些参数
            cb.add(fn1);
            cb.add(fn2);
            cb.fire() //执行调用
            cb.remove(fn2); //删除方法

        (3043,3183) Deffered:jquery中的延迟对象：对异步的统一管理。js中存在很多异步操作，如果不统一管理，代码会很复杂。
                            延迟的原理就是利用回调函数，先对回调函数进行存，当前面的异步操作执行完以后，进行回调函数的触发，再执行存储的回调函数。
                var dfd=$.Deffered();
                setTimeout(function(){
                    alert(1);
                    dfd.resolve();
                },1000);
                dfd.done(function(){
                    alert(2);
                })
                //先输出1，在输出2，让代码顺序执行

        (3184,3295) support :jQuery中的功能检测，不同的浏览器会有不同的js写法。通过功能检测，判断浏览器是否支持某些特性，新版本浏览器会和老版本浏览器存在着一些差异。
        
        (3308,3652) data()：jquery中的数据缓存。jquery中经常要对一些元素上的数据进行操作。其实数据并没有存储在元素上面，而是通过一些列的操作方法实现的。
                $(".ele").data("name","zhangsan");
                $(".ele").data("name");

        (3653,3797) queue:jQuery中队列管理方法。主要有queue()方法和unqueue()，在动画过程中经常使用入队和出队方法。
                $(".ele").animate({width:300});
                $(".ele").animate({height:200});
                $(".ele").animate({left:200});
                首先会将所有的存到队列中，执行队列的第一个操作时，调用出队操作时采用继续调用下一个方法，
                所以在函数或者动画中操作按照一定的顺序来执行是十分有效的
        
        (3803,4299) attr() prop() addClass() val():对元素属性的操作

        (4300,5128) on() trigger() 事件操作的相关方法，事件的添加，删除，事件委托等。

        (5140,6057) DOM操作方法：添加，删除，获取，包装方法，DOM的筛选

        (6058,6620) css方法：样式的操作。为什么有这么多行？会做很多兼容性，以及浏览器支持的处理，css样式支持对象，像素，百分比的处理。

        (6621,7854) 提交的数据和ajax操作。包括ajax()、load()、getJSON()方法等。

        (7855,8584) animate() :jQuery中动画的操作。包括一些封装好的方法show()、hide()

        (8585,8792) 位置与尺寸的一些方法。offset()

        (8804,8821) jQuery支持模块化的模式。不仅支持前端的AMD规范，还支持CommonJs规范

        (8826) window.jQuery=window.$=jQuery
     }
   )()使用匿名函数自执行来加载框架代码，jquery代码变为局部变量，不会在引用中和其他的变量名冲突

2. 对外提供变量接口的方法 window.$=$;

3.详细分析源码
    (1) (function(window,undefined){
                window.a=123;
        })(window,undefined);
        匿名函数内部可以直接使用window，为什么还要当做参数传进函数中呢？
            I 根据作用域链的查找原理，window在查找时会一层一层像上查找直到最外层，而当做参数传进来之后不需要查找了，能够提高查找效率。
            II 能够方便压缩，内部直接使用window时，无法进行压缩，压缩过后不能直到该变量就是window变量。
        为什么要传undefined?
            因为undefined是window下的一个属性，他可以在jquery外部被修改，如果不传undefined进来，外面修改的undefined的值会影响到里面的undefined

        为什么要定义rootjQuery 即jQuery(document) 代表文档对象？
            在内部会多次使用到，方便代码压缩，节省内存。

        为什么要定义变量core_strundefined = typeof undefined 变量?
            window.a==undefined;
            typeof window.a=="undefined";
            两种判断变量是否为undefined的方法都可以，但是第一种方法在IE8，9中使用xml时对节点进行判断时出现bug，所以用第二种方法来判断。将undfined赋值给变量也可以方便代码压缩
        
        jQuery中如何防止$变量冲突的问题？
            _jQuery = window.jQuery
            _$ = window.$
            当$被赋值时，window.$被其他库使用时，window.$的值肯定是除了undefined之外的其他值，如果没有被占用则就是undefined。这个在extend模块中有相应的方法进行判断。
        
        core_deletedIds = []变量为何没用了？
            数据缓存中用到的id值，但是在2.0.3版本之后采用对象形式来存储了，在之前的版本中是有用的，但2.0.3版本之后就没用了。
        
        jquery中链式调用的原理？
            jQuery = function(selector, context) {
                return new jQuery.fn.init(selector, context, rootjQuery);
            },
            jQuery.fn = jQuery.prototype={
                init:function(){

                }
            }
            jQuery.fn.init.prototype = jQuery.fn;
           jQuery()方法并不是一个构造函数，其返回值才是调用构造函数的过程。 init函数的原型对象是指向jquery的原型的。所有调用init方法后，都可以调用Jquery的方法。
        
        为什么要使用  rmsPrefix = /^-ms-/正则， rdashAlpha = /-([\da-z])/gi？
            margin-left:margin-left;
            -webkit-margin-left:webkitMarginLeft
            -ms-margin-left:MsMarginLeft;
            后者的则增是转换大小写 transform-2d：2d
        jquery中对css3的属性使用驼峰式转换，但是IE浏览器中比较特殊，所以需要单独定义一个正则匹配来进行区分。
    
    2.给jQuery对象，添加了一些方法和属性
        jQuery.fn=jQuery.prototype={
            jquery:版本,
            constructor:修正指向的问题,
            init():初始化参数和参数管理,
            selector:存储选择字符串,
            length:this对象的长度,
            toArray():转数组,
            get():转原生集合,
            pushStack:jQuery对象入栈,
            each():遍历集合
            ready(): DOM加载接口
            slice():集合的截取
            first():集合的第一项
            last():集合的最后一项
            eq():集合的指定项
            map(): 返回新的集合
            end():返回集合前一个状态
            push() (内部使用)
            sort(): (内部使用)
        }
        $("li")  $("li","ul")第二个参数代表执行上下文。不写的话就是默认document
        init()方法会对传入的参数进行一个分配。
            第一个判断先对一些不正确的参数进行处理，即$("") $(null) $(undefined) $(false)
            第二个判断，识别字符串
                字符串中又可以包含选择器，或者标签
                $("#div") $(".box") $("div") $("#div div .box");
                $('<div></div>')  $('<li>123</li><li>123</li>');
            第三个判断判断出this，document对象
            $(this) $(document)
            第四个判断，判断出传入的是函数
            $(function(){

            })
            第五个判断，判断为数组，或者对象
            $([]) $({})
        
        正则表达式的用法？
            量词：代表出现的次数
                {n,m}：至少出现n次，最多m次
                {n,} :至少n次
                * : 任意次 相当于{0,}
                ？ ：零次或一次 相当于{0,1}
                + ：一次或任意次相当于 {1,}
                {n}： 正好n次
                
                \s : 空格
                \S : 非空格
                \d : 数字
                \D : 非数字
                \w : 字符 ( 字母 ，数字，下划线_ )
                \W : 非字符
                .（点）——任意字符
                \. : 真正的点
                \b : 独立的部分 （ 起始，结束，空格 ）
                \B : 非独立的部分

        正则表达式中?:代表什么意思？
            这个代表不捕获分组比较(X)和(?:X)，前者是捕获分组，后者不捕获，区别在于正则表达式匹配输入字符串之后所获得的匹配的（数）组当中没有(?:X)匹配的部分；

        正则表达式中[^>]*代表什么意思？
            [^>]表示不为>的任意一个字符，且出现任意次。
            
        js中正则表达式的reg.exec(str)表示什么意思？
            正则表达式是以()来区分组的
            该方法是专门为捕获组而设计的。即返回一个匹配到的项的数组。如果没有匹配到时则返回的是一个null
            例如正则表达式 rquickExpr = /^(?:\s*(<[\w\W]+>)[^>]*|#([\w-]*))$/ 在匹配字符串str='<div>hello</div>'时
                返回的结果为 ['<div>hello','div',null],数组的第一项表示匹配到的项的字符信息，第二项表示匹配到的第一个组的信息，第三项表示匹配到的第二个组的信息，以此类推
           如正则表达式中没有捕获组，即只会返回匹配到的第一条信息,即使设置了全局匹配(g)，也只会返回第一个匹配项。
            例如 var reg=/.at/gi;var text='cat bat sat fat';
                var match=reg.exec(text);//返回值为cat
        
        $('li').css("background",'red')在jquery内部是如何实现的？

        var dom=document.querySelectorAll('li');
        for(var i=0;i<dom.length;i++){
            dom[0].style.background='red';
        }
        在jquery中也大致是按照上述方法来实现的，但是，jquery框架不可能知道用户输入的是什么元素，而且$("li")直接就生成了一个jquery对象，选择的元素会存在一个this对象中，结构如下
        this={
            0:'li',
            1:'li',
            2:'li',
            length:3
        }
        所以在jquery中可以直接访问到this对象,this[0]直接就代表的是原生dom对象
        for(var i=0;i<this.length;i++){
            this[0].style.background='red';
        }

        init方法中的判断逻辑是怎么样的呢？
            (1)当创建如下介个jquery对象时 $(""), $(null), $(undefined), $(false)
                if (!selector) {
                    return this;
                }
            (2)当输入的selector是字符串时,会出现创建标签，和选择器两种情况。这里都是通过math的值来进行进一步的判断
                   if (typeof selector === "string") {
                       //判断是否是标签
                        if (selector.charAt(0) === "<" && selector.charAt(selector.length - 1) === ">" && selector.length >= 3) {
                            match = [null, selector, null];
                            
                        } else {
                            //else中出现的情况会比较复杂又"<div>hello",但标签这种形式的，以及"#div"id形式的，以及其他类选择器形式的，
                            match = rquickExpr.exec(selector);
                            //当出现的是"<div>hello"形式的selector时，match=['<div>hello','<div>',null]
                            //当出现的时"#div"id形式的selector时，match=['#div',null,'div']
                            //当出现的是其他类型的选择器时，match=null
                        }
                        //根据match的值来进一步判断
                        //能够进入if条件句的只有单标签的形式以及#id的形式且id的形式没有指定上下文
                        if (match && (match[1] || !context)) {
                             if (match[1]) {
                                 //标签形式的处理
                             }else{
                                 //id形式的处理
                             }
                        }else if (!context || context.jquery) { 
                            //上下文不存在且上下文为jquery对象形式的处理$("ul",document) $("ul",$(document))
                             return (context || rootjQuery).find(selector);
                        }else{
                             return this.constructor(context).find(selector);
                        }
                   }else if (selector.nodeType) {
                       //处理$(this),$(document)等形式
                   }else if (jQuery.isFunction(selector)) {
                       //处理函数形式。$(function(){}),dom加载完成后立即执行，是$(document).ready(function(){})的简写形式
                        return rootjQuery.ready(selector);
                   }
                   //处理$($("#div"))的情况
                    if (selector.selector !== undefined) {
                        this.selector = selector.selector;
                        this.context = selector.context;
                    }
                    //处理$([]),$({})的情况
                    return jQuery.makeArray(selector, this);
        
        获取jquery元素时，返回的是一个this对象，如何将这个对象转换为数组呢？
            使用jQuery的实例方法toArray()即可做到。
            $("li").toArray();
            toArray()方法的实现原理是什么？
            调用数组的slice()执行环境为this对象。其中this对象的键值只能是0 1 2...才可以转换,且必须要有长度属性
            [].slice.call({0:"div",1:"div",length:2})
        
        如何理解jQuery实例中的get()方法？
            get()可以传参数，也可以不传参数。
            传参数代表获取对应的原生Dom，不传参数代表获取集合。其实就是调用了toArray方法。
            $("li").get();
        
        jquery中筛选选择器的实现原理是什么？
            利用的就是pushStack(),栈的原理，先进后出
            $("#div").eq(0) div进入栈底，第0个元素在栈顶。非栈底元素都会有一个prevObj属性指向下一个元素
