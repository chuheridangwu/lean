#  异步支持
关于异步操作，我们先来思考一下什么事情需要在子线程进行操作：
```markdown
1. 一些计算非常耗时的操作，会导致当前线程卡顿的操作
2. 一些需要等待的操作，比如文件的读取、服务器的相应等操作、或者等一个固定的时间。
```

在OC中，我们说的异步操作一般是指直接开辟一条子线程，把当前操作放到子线程中执行。而在dart中，大部分的异步操作是不需要开辟子线程的,类似于JavaScript，在一个线程中执行操作。

在Dart中，每一个线程都被封装在 Isolate 中，线程之间不能共享内存,通信只能通过发消息的方法。这样的隔离好处是既然不能共享变量，也就不会存在多线程中的抢票、存钱的问题。另外因为相互之间比较独立，所以垃圾回收机制效率也比较高。

一个线程是如何解决网络操作或者等待用户输入的情况呢？这里就要用到Event Loop。
## Event Loop / Event Queue
Event Loop 类似于 iOS中的Run Loop机制。一种循环机制，不停的检查 Event Queue 事件队列中是否存在事件，如果存在事件，则先处理当前事件，如果没有，则结束当前程序。

Dart中还有一个 Microtask Queue 事件队列,类似于VIP通道，如果当前队列中存在事件，则优先处理当前队列中的时间。在 Microtask Queue 事件队列里的时间处理完之后，才会处理 Event Queue 事件队列。`scheduleMicrotask(()=>print("C"));`表示向 Microtask Queue 队列中添加事件。
![](../imgs/flutter_img_2.jpg ':size=400')
![](../imgs/flutter_img_3.jpg ':size=300')

> 单线程机制只能解决等待的问题，如果是一些计算量太大耗时造成的操作，只能通过创建多个Isolate来解决。


Dart类库有非常多的返回`Future`或者`Stream`对象的函数。 这些函数被称为异步函数：它们只会在设置好一些耗时操作之后返回，比如像 IO操作。而不是等到这个操作完成。

`async`和`await`关键词支持了异步编程，允许您写出和同步代码很像的异步代码。

## Future
`Future` 与 JavaScript 中的 `Promise` 非常相似，表示一个异步操作的最终完成（或失败）及其结果值的表示。简单来说，它就是用于处理异步操作的，异步处理成功了就执行成功的操作，异步处理失败了就捕获错误或者停止后续操作。一个 Future 只会对应一个结果，要么成功，要么失败。

Future 的所有API的返回值仍然是一个 Future对象，所以可以很方便的进行链式调用。

### Future.then
为了方便示例，在本例中我们使用Future.delayed 创建了一个延时任务（实际场景会是一个真正的耗时任务，比如一次网络请求），即2秒后返回结果字符串"hi world!"，然后我们在then中接收异步结果并打印结果，代码如下：

```dart
Future.delayed(new Duration(seconds: 2),(){
   return "hi world!";
}).then((data){
   print(data);
});
```
### Future.catchError
如果异步任务发生错误，我们可以在 `catchError` 中捕获错误，我们将上面示例改为：
```dart
Future.delayed(new Duration(seconds: 2),(){
   //return "hi world!";
   throw AssertionError("Error");  
}).then((data){
   //执行成功会走到这里  
   print("success");
}).catchError((e){
   //执行失败会走到这里  
   print(e);
});
```

在本示例中，我们在异步任务中抛出了一个异常，then的回调函数将不会被执行，取而代之的是 `catchError` 回调函数将被调用；但是，并不是只有 catchError 回调才能捕获错误，then方法还有一个可选参数`onError`，我们也可以它来捕获异常：
```dart
Future.delayed(new Duration(seconds: 2), () {
	//return "hi world!";
	throw AssertionError("Error");
}).then((data) {
	print("success");
}, onError: (e) {
	print(e);
});
```

### Future.whenComplete  完成后回调

有些时候，我们会遇到无论异步任务执行成功或失败都需要做一些事的场景，比如在网络请求前弹出加载对话框，在请求结束后关闭对话框。这种场景，有两种方法，第一种是分别在then或catch中关闭一下对话框，第二种就是使用Future的whenComplete回调，我们将上面示例改一下：

```dart
Future.delayed(new Duration(seconds: 2),(){
   //return "hi world!";
   throw AssertionError("Error");
}).then((data){
   //执行成功会走到这里 
   print(data);
}).catchError((e){
   //执行失败会走到这里   
   print(e);
}).whenComplete((){
   //无论成功或失败都会走到这里
});
```

### Future.wait

有些时候，我们需要等待多个异步任务都执行结束后才进行一些操作。

比如我们有一个界面，需要先分别从两个网络接口获取数据，获取成功后，我们需要将两个接口数据进行特定的处理后再显示到UI界面上，应该怎么做？答案是`Future.wait`，

它接受一个`Future`数组参数，只有数组中所有`Future`都执行成功后，才会触发then的成功回调，只要有一个`Future`执行失败，就会触发错误回调。下面，我们通过模拟`Future.delayed` 来模拟两个数据获取的异步任务，等两个异步任务都执行成功时，将两个异步任务的结果拼接打印出来，代码如下：

```dart
Future.wait([
  // 2秒后返回结果  
  Future.delayed(new Duration(seconds: 2), () {
    return "hello";
  }),
  // 4秒后返回结果  
  Future.delayed(new Duration(seconds: 4), () {
    return " world";
  })
]).then((results){
  print(results[0]+results[1]);
}).catchError((e){
  print(e);
});
```
执行上面代码，4秒后你会在控制台中看到“hello world”。

## Async/await
Dart中的 `async/await` 和 JavaScript中的`async/await`功能和用法是一模一样的，如果你已经了解JavaScript中的async/await的用法，可以直接跳过本节。

`async/await`都只是一个语法糖，编译器或解释器最终都会将其转化为一个`Promise（Future）`的调用链。

### 回调地狱(Callback Hell)
如果代码中有大量异步逻辑，并且出现大量异步任务依赖其它异步任务的结果时，必然会出现`Future.then`回调中套回调情况。

例如: 现在有个需求场景是用户先登录，登录成功后会获得用户ID，然后通过用户ID，再去请求用户个人信息，获取到用户个人信息后，为了使用方便，我们需要将其缓存在本地文件系统，代码如下：

```dart
//先分别定义各个异步任务
Future<String> login(String userName, String pwd){
	...
    //用户登录
};
Future<String> getUserInfo(String id){
	...
    //获取用户信息 
};
Future saveUserInfo(String userInfo){
	...
	// 保存用户信息 
}; 
```

接下来，执行整个任务流：
```dart
login("alice","******").then((id){
 //登录成功后通过，id获取用户信息    
 getUserInfo(id).then((userInfo){
    //获取用户信息后保存 
    saveUserInfo(userInfo).then((){
       //保存用户信息，接下来执行其它操作
        ...
    });
  });
})
```
可以感受一下，如果业务逻辑中有大量异步依赖的情况，将会出现上面这种在回调里面套回调的情况，过多的嵌套会导致的代码可读性下降以及出错率提高，并且非常难维护，这个问题被形象的称为**回调地狱（Callback Hell**）。

回调地狱问题在之前 JavaScript 中非常突出，但随着ECMAScript6和ECMAScript7标准发布后，这个问题得到了非常好的解决，而解决回调地狱的两大神器正是引入了 `Promise`，以及 ECMAScript7 中引入的`async/await`。

Dart中几乎是完全平移了JavaScript中的这两者：`Future` 相当于 `Promise`，而`async/await`连名字都没改。接下来我们看看通过`Future`和`async/await`如何消除上面示例中的嵌套问题。

#### 使用Future消除Callback Hell
```dart
login("alice","******").then((id){
  	return getUserInfo(id);
}).then((userInfo){
    return saveUserInfo(userInfo);
}).then((e){
   //执行接下来的操作 
}).catchError((e){
  //错误处理  
  print(e);
});
```

正如上文所述， “`Future` 的所有API的返回值仍然是一个`Future对象`，所以可以很方便的进行链式调用” ，如果在then中返回的是一个Future的话，该future会执行，执行结束后会触发后面的then回调，这样依次向下，就避免了层层嵌套。

#### 使用async/await 消除callback hell
通过Future回调中再返回Future的方式虽然能避免层层嵌套，但是还是有一层回调，有没有一种方式能够让我们可以像写同步代码那样来执行异步任务而不使用回调的方式？答案是肯定的，这就要使用`async/await`了，下面我们先直接看代码，然后再解释，代码如下：

```dart
task() async {
   try{
    String id = await login("alice","******");
    String userInfo = await getUserInfo(id);
    await saveUserInfo(userInfo);
    //执行接下来的操作   
   } catch(e){
    //错误处理   
    print(e);   
   }  
}
```

`async`用来表示函数是异步的，定义的函数会返回一个`Future对象`，可以使用`then`方法添加回调函数。

`await` 后面是一个`Future`，表示等待该异步任务完成，异步完成后才会往下走；`await`必须出现在 `async 函数内部`。
可以看到，我们通过`async/await`将一个异步流用同步的代码表示出来了。

其实，无论是在JavaScript还是Dart中，`async/await`都只是一个语法糖，编译器或解释器最终都会将其转化为一个`Promise（Future）`的调用链。

## 推荐阅读 
* [Flutter实战电子书](https://book.flutterchina.club/chapter1/dart.html#_1-4-3-%E5%BC%82%E6%AD%A5%E6%94%AF%E6%8C%81)