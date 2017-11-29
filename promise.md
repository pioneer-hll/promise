# 什么是promise
promise就是个构造函数，参数为一个函数，使用样式为：
```
var p = new Promise(function(resolve,reject){
    console.log("结果");
    resolve("结果")
    })
/*p.then(function(data){
    console.log(data)
    })*/
```

可以看出来 只要一 new ，promise就执行了.


+ 这玩意有什么作用呢？
+ 怎么用呢？
根据作用需求来学，
比方一个页面加载进来，需要发送5个ajax请求，并需要把结果统一经行处理：
+ 首先大家按照大家已有的知识点来思考一下如何解决呢？
代码如下：

~~~
$(function(){
            var arr = []
            $.get("https://cnodejs.org/api/v1/topics?tab=ask",function(data){
                arr.push(data);
                $.get("https://cnodejs.org/api/v1/topics?tab=job",function(data){
                    arr.push(data);
                    $.get("https://cnodejs.org/api/v1/topics?tab=good",function(data){
                        arr.push(data);
                        $.get("https://cnodejs.org/api/v1/topics?tab=share",function(data){
                            arr.push(data);
                                console.log(arr)
                        })
                    })
                
                })
            })
        })
~~~
简称 回调地狱 

创新一下，代码如下：
~~~
    (function () {
      var count = 0;
      var arr = [];

      $.get("https://cnodejs.org/api/v1/topics?tab=good",function(data){
        arr.push(data);
        count++;
        handle()
        })
      $.get("https://cnodejs.org/api/v1/topics?tab=job",function(data){
        arr.push(data);
        count++;
        handle()
        })
      $.get("https://cnodejs.org/api/v1/topics?tab=share",function(data){
        arr.push(data);
        count++;
        handle()
        });
      $.get("https://cnodejs.org/api/v1/topics?tab=ask",function(data){
        arr.push(data);
        count++;
        handle()
        });

      function handle() {
        if (count === 4) {
          console.log(arr);
        }
      }
    })();
~~~

但是这种依然有个缺点，得写监控函数，每次回调都会调用监控函数，耗费性能，还有其他方法吗？


*Promise* 来了，用promise怎么实现呢？


代码如下：
~~~
        $(function(){
            // 封装一个promise;
            var  p  = function(url){
                return new Promise(function(resolve,reject){
                    $.get(url,function(data){
                    resolve(data);
                    })
                })
            }
            Promise.all([
                    p("https://cnodejs.org/api/v1/topics?tab=good"),
                    p("https://cnodejs.org/api/v1/topics?tab=share"),
                    p("https://cnodejs.org/api/v1/topics?tab=ask"),
                    p("https://cnodejs.org/api/v1/topics?tab=job"),
                ]).then(function(result){
                    console.log(result);
                })
        })
~~~

如果顺序调用呢？

~~~
        $(function(){
            // 封装一个promise;
            var  p  = function(url){
                return new Promise(function(resolve,reject){
                    $.get(url,function(data){
                    resolve(data);
                    })
                })
            }

            var arr = []
            p("https://cnodejs.org/api/v1/topics?tab=ask")
            .then(function(data){
                arr.push(data);
                return p("https://cnodejs.org/api/v1/topics?tab=share")
            }).then(function(data){
                arr.push(data);
                return p("https://cnodejs.org/api/v1/topics?tab=ask")
            }).then(function(data){
                arr.push(data);
                return p("https://cnodejs.org/api/v1/topics?tab=good")
            }).then(function(data){
                arr.push(data);
                console.log(arr);
            })
                
            })
~~~

再看一个应用场景：
请求某个资源，固定时间内没有返回的话，渲染默认数据：


这应用到一个race接口，将4.html的js代码改成如下：
~~~
    <script>
        $(function(){
            // 封装一个promise;
            var  p  = function(url){
                return new Promise(function(resolve,reject){
                    $.get(url,function(data){
                    resolve(data);
                    })
                })
            }
            Promise.race([
                    p("https://cnodejs.org/api/v1/topics?tab=good"),
                    p("https://cnodejs.org/api/v1/topics?tab=share"),
                    p("https://cnodejs.org/api/v1/topics?tab=ask"),
                    p("https://cnodejs.org/api/v1/topics?tab=job"),
                ]).then(function(result){
                    console.log(result);
                })
        })
    </script>
~~~

运行看结果，谁最先返回就打印谁的数据：
~~~
$(function(){
            // 封装一个promise;
            var  p  = function(url){
                return new Promise(function(resolve,reject){
                    $.get(url,function(data){
                    resolve(data);
                    })
                })
            }

            // var p2 = function(){
            //  return new Promise(function(resolve,reject){
            //      reject("err");
            //  })
            // }

            // P2的另一种形态:
            // 如果是reject会直接跳到catch里面去，如果是resolve,会接着执行，不会跳跃
            var p2 = function(){
                return new Promise(function(resolve,reject){
                    setTimeout(function(){
                        reject("err");
                    },100)
                })
            }

            Promise.race([
                    p("https://cnodejs.org/api/v1/topics?tab=good"),
                    p2()
                ]).then(function(result){
                    console.log(result);
                }).then(function(){
                    console.log("我运行了")
                }).catch(function(data){
                    console.log(data);
                })
        })
~~~