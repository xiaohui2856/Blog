---
title: JavaScript：定时器中的异步请求堆积如何解决？
date: 2020-03-14 12:50:34
categories: JavaScript
tags: JavaScript
keywords: 异步刷新, 轮询
---


## 说明

有时候我们需要用setinterval来执行一些异步操作来刷新页面的信息。但是如果一旦异步函数的请求时间过长，就会造成事件操作堆积。

如：

```

this.interval = setInterval(async () =# {
      const resp = await ajax();
      this.rows = resp.rows;
    }, 5000);

```
假设ajax()请求的时间因为网络或者其他原因超过了5秒，定时器并不会等待其返回结果，而是依旧会进行下一次循环，这样的话，其实定时器和异步请求之间的时序并没有联系顺序也毫无规律，那么我们如何实现等本次ajax请求完毕之后等待一段时间之后再进行下一次循环呢？

<!-- more -->
### 1.简单代码实现

```
    function setIntervalWaitable(callback,ms){
        this._self = {
            fn: callback,
            timeout: ms,
            timeoutHandler: null
        }
    }
    setIntervalWaitable.prototype.request = function() {
        if (this._self.timeoutHandler) {
            clearTimeout(this._self.timeoutHandler);
        }

        this._self.timeoutHandler = setTimeout(() => {
            this._self.fn();
        }, this._self.timeout);
    }

    <!-- 模拟执行 -->
    <!-- let ms = 2000;
    let asyncMs = 1000;
    var set = new setIntervalWaitable(()=>{
        setTimeout(function(){
            set.request()
        }, asyncMs)
    }, ms)
    set.request() -->

```

达到如下效果的异步刷新时序图：

```
rfn########----------------rfn##----------------rfn########################----------------rfn####
-----------5000ms##########-----5000ms##########---------------------------5000ms##########
```


### 2. 改造：
  * 等待传入的ms时间，如果此时callback已经完成，重新执行callback
  * 否则，等待callback完成，再重新执行callback
  
例如500ms周期时序图:
rpc########----rpc############rpc########################rpc####--------rpc######------rpc####
500ms##########500ms##########500ms##########------------500ms##########500ms##########

#### 实现思路：

在第一题的基础上，在异步函数调用settimeout之前判断异步函数用时和settimeout的时间大小，计算差值，以差值作为下次定时器执行的时间。 如果异步函数用时大于定时器设定时间，直接执行下次异步函数即可

#### 代码实现
```

    function setIntervalWaitable2(callback,ms){
        this._self = {
            fn: callback,
            timeout: ms,
            timeoutHandler: null,
            start: null,
            duration: null
        }
    }
    setIntervalWaitable2.prototype.request = function() {
        let that = this;
        let timeout = 0; //首次直接执行事件

        if (that._self.timeoutHandler) {
            clearTimeout(that._self.timeoutHandler);
        }
        // 判断
        if (that._self.start) { //计算更新timeout值
            that._self.duration = new Date().getTime() - that._self.start;
            if (that._self.duration < that._self.timeout) {
                timeout = that._self.timeout - that._self.duration;
            }
        }
        that._self.timeoutHandler = setTimeout(() => {
            that._self.start = new Date().getTime();
            that._self.fn();
        }, timeout);
    }

    <!-- 模拟执行
    let ms = 2000;
    let asyncMs = 1000; //假设异步操作时间

    var set = new setIntervalWaitable2(()=>{
        setTimeout(function(){
            console.log('函数执行' + new Date().getSeconds())
            set.request()
        }, asyncMs)
    }, ms)
    set.request()  -->

    <!-- 实际执行 -->

     <!-- 实际执行 -->
    created() {
          this.interval = new setIntervalWaitable2(()=>{
              const resp = await getNewStatistics();
              this.rows = resp.rows;
              this.interval.request()
          }, 500)
          this.interval.request() //执行
      }

```


### 3. 实现暂停和重启


#### 实现思路:

增加flag判断即可。

#### 代码
```
    function setIntervalWaitable3(callback,ms){
        this._self = {
            fn: callback,
            timeout: ms,
            timeoutHandler: null,
            timeoutEnabled: true,
            start: null,
            duration: null
        }
    }
    setIntervalWaitable3.prototype.request = function() {
        let that = this;
        let timeout = 0;

        if (!that._self.timeoutEnabled) {
            return;
        }

        if (that._self.timeoutHandler) {
            clearTimeout(that._self.timeoutHandler);
        }

        that._self.duration = new Date().getTime() - that._self.start;

        if (that._self.duration < that._self.timeout) {
            timeout = that._self.timeout - that._self.duration;
        }
        that._self.timeoutHandler = setTimeout(() => {
            that._self.start = new Date().getTime();
            that._self.fn();
        }, timeout);
    }

    setIntervalWaitable3.prototype.stop = function() {
        this._self.timeoutEnabled = false;
    }


    setIntervalWaitable3.prototype.restart = function() {
        this._self.timeoutEnabled = true;
		this.request()
    }

    <!-- 模拟执行 -->
    <!-- let ms = 2000;
    let asyncMs = 1000; //假设异步操作时间

    var set = new setIntervalWaitable3(()=>{
        setTimeout(function(){
            console.log('函数执行' + new Date().getSeconds())
            set.request()
        }, asyncMs)
    }, ms)
    set.request() //执行 -->


    <!-- 实际执行 -->

    created() {
        this.interval = new setIntervalWaitable3(()=>{
            const resp = await getNewStatistics();
            this.rows = resp.rows;
            this.interval.request()
        }, 500)
        this.interval.request() //执行
    }
    destroyed() {
        this.interval.stop();
    }
```
## 总结
如上实现过程，思路很简单，就是在异步请求结束后手动调起下次循环，这样才能保证时序性。（当然我现在也只能想到这种办法，无论怎么考虑，第二次循环始终需要异步请求结尾处调用。）

目前我已经将该功能进行了封装，对其功能进行了一部分添加和完善。包括两种循环模式。



> 详情  [Github 项目]("https://github.com/lunhui1994/async-loop-timer")

> npm 使用

> npm i async-loop-timer