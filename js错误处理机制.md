# 深入理解JavaScript错误处理机制

## 1. 错误分类

JavaScript错误，可分为编译错误、运行时错误、资源加载错误。

### 1.1 js运行时错误

JavaScript提供了一种捕获运行时错误的捕获机制。如果代码能够捕获潜在的错误，并能适当处理，就能确保代码不会再运行时产生意向不到的错误，给用户造成困扰，这也意味着代码的质量是非常高的。

#### 1.1.1  Error实例对象

JavaScript解析或运行时，一旦发生错误，引擎就会抛出一个错误对象。JavaScript原生提供Error构造函数，所有抛出的错误都是这个构造函数的实例。

Error实例对象的三个属性：

- message 错误提示信息
- name 错误名称
- stack 错误的堆栈

例如下面的代码，打印错误实例对象，可以得到message name stack信息：

```js
var err = new Error('出错了！');
console.dir(err);
```

![1553790823862](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1553790823862.png)

上面的例子中，err是一个对象（object）类型，拥有message、stack两个属性，还有一个原型链上的属性name，来自于构造函数Error的原型。

#### 1.1.2  6种错误类型

以下6中错误类型都是Error对象的派生对象。在JavaScript中，数组array、函数function都是特殊的对象：

1. SyntaxError语法错误

   SyntaxError是代码解析式发生的语法错误。例如，写了一个错误的语法var a = 

   ![1554095622433](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554095622433.png)

2. TypeError类型错误

   TypeError是变量或者参数不是预期类型时发生的错误。例如在number类型上调用array的方法。

   ![1554095864562](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554095864562.png)

3. RangeError范围错误

   RangeError是一个值超过有效范围发生的错误。例如设置数组的长度为一个负值。

   ![1554095921373](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554095921373.png)

4. ReferenceError引用错误

   ReferenceError是引用一个不存在的变量时发生的错误。

   ![1554095979207](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554095979207.png)

5. EvalError eval错误

   eval函数没有被正确执行时，会抛出EvalError错误。该错误类型已经不再使用了，只是为了保证与以前代码兼容，才继续保留。

   ![1554096138634](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554096138634.png)

6. URIError URL错误

   URIError指调decodeURI encodeURI decodeURIComponent encodeURIComponent escape UNescape时发生的错误。

   ![1554096290713](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554096290713.png)

###  1.2  资源加载错误

当以下标签（不包含<link>），加载资源出错时，会发生资源加载错误。

![1554096798055](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554096798055.png)

资源加载错误可以用onerror事件监听。

![1554096845312](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554096845312.png)

资源加载错误不会冒泡，只能在事件流捕获阶段获取错误。

![1554096889350](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554096889350.png)

当加载跨域资源时，不会报错，需要在元素上添加crossorigin，同时服务器需要在responseheader中，设置Access-Control-Allow-Origin为*获取允许的域名。

![1554096976718](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554096976718.png)

## 2  错误捕获

参考阿里开源框架jstracker源码。

![1554097072888](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554097072888.png)

上面的代码，不是很严谨，如果用户在代码中也写了window.onerror，会被覆盖，导致错误没有正常上报。

## 3  throw

MDN关于throw的定义

throw语句用来抛出一个用户自定义的异常。当前函数的执行将被停止（throw之后的语句将不会执行），并且控制将被传递到调用堆栈中的第一个catch块。如果调用者函数中没有catch块，程序将会终止。

“throw之后的语句将不会执行”，这句话比较容易理解，例如：

![1554097383546](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554097383546.png)

“如果调用者函数中没有catch块，程序将会终止”，这句话是有问题的。下面用代码来推翻这个结论：

![1554097439609](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554097439609.png)

运行上面的代码，控制台首先会抛出错误，然后每秒打印“setInterval依然在执行”。

![1554097546837](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554097546837.png)

点击btn-1，打印1；点击btn-2，无反应。

这就说明：throw之后，程序没有停止运行。

结论：throw之后的语句不会执行，并且控制将被传递到调用堆栈中的第一个catch块。如果调用者函数中没有catch块，程序也不会停止，throw之前的语句依旧在执行。

## 4  try...catch...finally

try/catch的作用是将可能引发错误的代码放在try块中，在catch中捕获错误，对错误进行处理，选择是否往下执行。

### 4.1  try代码块中的错误，会被catch捕获，如果没有手动抛出错误，不会window捕获

![1554097849874](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554097849874.png)

catch中抛出异常，用throw e，不要用throw new Error（e），因为e本身就是一个Error对象了，具有错误的完成堆栈信息stack，new Error会改变堆栈信息，将堆栈定位到当前这一行。

### 4.2  try...finally...不能捕获错误

下面的代码，由于没有catch，错误会直接被window捕获。

![1554098490124](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554098490124.png)

### 4.3  try...catch...只能捕获同步代码的错误，不能捕获异步代码错误

下面的代码，错误将不能被catch捕获。

![1554098766527](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554098766527.png)

因为setTimeout是异步任务，里面回调函数会被放入到宏任务队列中，catch中代码块属于同步任务，处于当前的事件队列中，会立即执行。

当setTimeout中回调执行时，try/catch中代码块已经不在堆栈中。所以错误不能被捕获。

## 5  promise

Promise对象是JavaScript的一种异步操作解决方案。Promise是构造函数，也是对象。

promise的三种状态：

- pending异步操作为完成
- fulfilled异步操作成功
- rejected异步操作失败

如果一个promise没有resolve或reject，将一直处于pending状态。

### 5.1  Promise的两个方法

- Promise.prototype.then 通常用来添加异步操作成功的回调
- Promise.prototype.catch 用来添加异步操作失败的回调

### 5.2  Promise内部的错误捕获

用promise可以解决“回调地狱“的问题，但如果不能处理好Promise错误，将会陷入另一个地狱：错误将被“吞掉”，可能不会在操作台打印，也不会被window捕获。给调试、线上故障排查带来很大困难。

Promise内部抛出的错误，都不会被window捕获，除非用了setTimeout/setInterval。

例子1，错误会抛出到控制台，promise.catch回调能够执行，但错误不会被window捕获。

![1554101147197](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554101147197.png)

例子2，p.then中当回调函数出错，错误会抛出到控制台，promise.catch回调能够执行，但错误不会被window捕获。

![1554101222408](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554101222408.png)

例子3，p.catch回调出错，错误会抛出到控制台，后续的promise.catch回调能够执行，但错误不会被window捕获。

![1554101282167](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554101282167.png)

例子4，错误会抛出到控制台，后续的promise.catch回调不会执行，错误会被window捕获。

![1554101344023](C:\Users\jxn\AppData\Roaming\Typora\typora-user-images\1554101344023.png)

例3和例4完全不一样的结果，为什么会这样呢？因为promise内部也实现了类似于try/catch的错误捕获机制，能够捕获错误。

### 5.3  在全局捕获promise错误

#### 5.3.1  unhandledrejection捕获未处理Promise错误

unhandledrejection时间在浏览器中兼容性不好，通常不这么做。

## 6  async/await

当调用一个async函数时，会返回一个promise对象。当这个async函数返回一个值时，promise的resolve方法会负责传递这个值；当async函数抛出异常时，promise的reject方法也会传递这个异常值。

async/await的用途是简化使用promise异步调用的操作，并对一组Promises执行某些操作。正如promises类似于结构化回调，async/await类似于组合生成器和promises。

