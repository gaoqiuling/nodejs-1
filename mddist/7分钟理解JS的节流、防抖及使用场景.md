前言
==

据说阿里有一道面试题就是谈谈**函数节流**和**函数防抖**。  
糟了，这可触碰到我的知识盲区了，好像听也没听过这2个东西，痛定思痛，赶紧学习学习。here we go！

![15341407332107](/img/remote/1460000016261605?w=300&h=331 "15341407332107")

概念和例子
=====

### 函数防抖(debounce)

> 在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。

看一个🌰（栗子）：

    //模拟一段ajax请求
    function ajax(content) {
      console.log('ajax request ' + content)
    }
    
    let inputa = document.getElementById('unDebounce')
    
    inputa.addEventListener('keyup', function (e) {
        ajax(e.target.value)
    })
    

看一下运行结果：

![2018-09-04 09 23 46](https://static.segmentfault.com/v-5b973eab/global/img/squares.svg "2018-09-04 09 23 46")

可以看到，我们只要按下键盘，就会触发这次ajax请求。不仅从资源上来说是很浪费的行为，而且实际应用中，用户也是输出完整的字符后，才会请求。下面我们优化一下：

    //模拟一段ajax请求
    function ajax(content) {
      console.log('ajax request ' + content)
    }
    
    function debounce(fun, delay) {
        return function (args) {
            let that = this
            let _args = args
            clearTimeout(fun.id)
            fun.id = setTimeout(function () {
                fun.call(that, _args)
            }, delay)
        }
    }
        
    let inputb = document.getElementById('debounce')
    
    let debounceAjax = debounce(ajax, 500)
    
    inputb.addEventListener('keyup', function (e) {
            debounceAjax(e.target.value)
        })

看一下运行结果：

![2018-09-04 09 29 50](https://static.segmentfault.com/v-5b973eab/global/img/squares.svg "2018-09-04 09 29 50")

可以看到，我们加入了防抖以后，当你在频繁的输入时，并不会发送请求，只有当你在指定间隔内没有输入时，才会执行函数。如果停止输入但是在指定间隔内又输入，会重新触发计时。  
再看一个🌰：

        
    let biu = function () {
        console.log('biu biu biu',new Date().Format('HH:mm:ss'))
    }
    
    let boom = function () {
        console.log('boom boom boom',new Date().Format('HH:mm:ss'))
    }
    
    
    setInterval(debounce(biu,500),1000)
    setInterval(debounce(boom,2000),1000)

看一下运行结果：

![2018-09-04 09 32 21](https://static.segmentfault.com/v-5b973eab/global/img/squares.svg "2018-09-04 09 32 21")

这个🌰就很好的解释了，如果在时间间隔内执行函数，会重新触发计时。biu会在第一次1.5s执行后，每隔1s执行一次，而boom一次也不会执行。因为它的时间间隔是2s，而执行时间是1s，所以每次都会重新触发计时

> ### 个人理解 函数防抖就是法师发技能的时候要读条，技能读条没完再按技能就会重新读条。

### 函数节流(throttle)

> 规定在一个单位时间内，只能触发一次函数。如果这个单位时间内触发多次函数，只有一次生效。

看一个🌰：

    
      function throttle(fun, delay) {
            let last, deferTimer
            return function (args) {
                let that = this
                let _args = arguments
                let now = +new Date()
                if (last && now < last + delay) {
                    clearTimeout(deferTimer)
                    deferTimer = setTimeout(function () {
                        last = now
                        fun.apply(that, _args)
                    }, delay)
                }else {
                    last = now
                    fun.apply(that,_args)
                }
            }
        }
    
        let throttleAjax = throttle(ajax, 1000)
    
        let inputc = document.getElementById('throttle')
        inputc.addEventListener('keyup', function(e) {
            throttleAjax(e.target.value)
        })

看一下运行结果：

![2018-09-04 09 36 49](https://static.segmentfault.com/v-5b973eab/global/img/squares.svg "2018-09-04 09 36 49")

可以看到，我们在不断输入时，ajax会按照我们设定的时间，每1s执行一次。

结合刚刚biubiubiu的🌰：

    
        let biubiu = function () {
            console.log('biu biu biu', new Date().Format('HH:mm:ss'))
        }
    
        setInterval(throttle(biubiu,1000),10)
    

![2018-09-04 09 37 58](https://static.segmentfault.com/v-5b973eab/global/img/squares.svg "2018-09-04 09 37 58")

不管我们设定的执行时间间隔多小，总是1s内只执行一次。

> ### 个人理解 函数节流就是fps游戏的射速，就算一直按着鼠标射击，也只会在规定射速内射出子弹。

总结
==

*   函数防抖和函数节流都是防止某一时间频繁触发，但是这两兄弟之间的原理却不一样。
*   函数防抖是某一段时间内只执行一次，而函数节流是间隔时间执行。

### 结合应用场景

*   debounce
    
    *   search搜索联想，用户在不断输入值时，用防抖来节约请求资源。
    *   window触发resize的时候，不断的调整浏览器窗口大小会不断的触发这个事件，用防抖来让其只触发一次
*   throttle
    
    *   鼠标不断点击触发，mousedown(单位时间内只触发一次)
    *   监听滚动事件，比如是否滑到底部自动加载更多，用throttle来判断

拓展
==

> 参考链接：[http://www.cnblogs.com/zichi/...](http://www.cnblogs.com/zichi/p/5331426.html)

![15343043539670](https://static.segmentfault.com/v-5b973eab/global/img/squares.svg "15343043539670")

这是高程中的经典代码：

    
        function throttle(method, context) {
            clearTimeout(method.tId);
            method.tId = setTimeout(function () {
                method.call(context);
            }, 100)
        }
    

我们通过上面的例子知道，其实这段函数应该是debounce函数防抖，而不是函数节流，很多文章也都会拿这段代码来做例子，函数本身没错，但是命名错了。

原作者的这段话就写的很好，

> 就以 throttle 为例，某日，老师给你布置了一个作业，让你深入理解一下 throttle，第二天上课来聊聊。张三心里非常高兴，这个概念在经典书籍《JavaScript高级程序设计》中见过，打开一看，就两页，而且解释地非常清晰，看完就高兴地干别的事情去了。而李四，觉得高程三讲的有点少，而去谷歌了下其他关于 throttle 的知识点，兴奋地看到 throttle 函数的好几种写法，发现高程三只是用了最简单的方式，还有更优雅运用场景更多的写法，或许此时他已经发现和 throttle 同时出现的还有个 debounce，这是什么鬼？反正老师没说，以后再看吧，于是心满意足地玩游戏去了。而王五，和李四一样发现了 debounce，这是什么？一起了解了吧，继而发现 debounce 的用法居然和高程三中的 throttle 一样！继续挖下去，发现高程三中的 throttle 函数其实应该叫 debounce，看到最后，王五已经把 throttle 和 debounce 彻底理解了。

我们要做王五，并且争取早日产出一手知识！加油！

广而告之
----

本文发布于[薄荷前端周刊](https://github.com/BooheeFE/weekly)，欢迎Watch & Star ★，转载请注明出处。

### 欢迎讨论，点个赞再走吧 ｡◕‿◕｡ ～

[](#awakeapp)Awake.app
======================

[中文介绍](http://type.so/object-c/awake-app.html)

[![screenshot](https://camo.githubusercontent.com/2bb97b776db20dbac366727cf21328d8f8a2c8bb/68747470733a2f2f7261772e6769746875622e636f6d2f7869616f7a692f4177616b652e6170702f6d61737465722f73637265656e73686f742e706e67)](https://camo.githubusercontent.com/2bb97b776db20dbac366727cf21328d8f8a2c8bb/68747470733a2f2f7261772e6769746875622e636f6d2f7869616f7a692f4177616b652e6170702f6d61737465722f73637265656e73686f742e706e67)

An app for mac osx to prevent sleeping; inspired by Caffeine.

1.  left click to toggle "should sleep status"
2.  right click for menu

"run at login" will not take effect unless you put this app in the `/Applications` directory.

### [](#download)download

[https://github.com/xiaozi/Awake.app/releases](https://github.com/xiaozi/Awake.app/releases)

### [](#apple-script-supported)apple script supported

tell application "Awake"
    turn on for 1
    \-\- turn off
end tell