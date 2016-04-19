###RxJava
####一.   一些概念
1. **ObServer（接口）** 相当于 OnclickListener，即观察者，有三个回调方法  onError()  onNext() onComplete；

	**ObServerble** 相当于View ，即被观察者；
		
	**subscribe** 订阅，通过subscribe()方法，将Observer和Observerble关联到一起

  **Subscriber** 是对ObServer的封装和扩展，使用方法完全一致，区别在于两点：
  1. Subscriber扩展了 onStart()
  2. unsubscribe() , 也是其扩展的方法，用于取消订阅
  
  **Subscription**  是一个接口，Subscriber实现了它

####二. 一些接口
1. **from(T[])** **just(T...)** obServable可以直接像这样去创建：Observable.from()  或者  ObServable.just()，这种创建方式等价于Observable.create()方式，只是传参方式不同；
2. **Action0 Action1**  Action0 是 RxJava 的一个接口，它只有一个方法 call()，这个方法是无参无返回值的；由于 onCompleted() 方法也是无参无返回值的，因此 Action0 可以被当成一个包装对象，将 onCompleted() 的内容打包起来将自己作为一个参数传入 subscribe() 以实现不完整定义的回调。这样其实也可以看做将 onCompleted() 方法作为参数传进了 subscribe()，相当于其他某些语言中的『闭包』。 
3. **Func1<String,Bitmap>** 和Action1非常相似，也是一个接口，但区别在于，Func1是有返回值的，和ActionX一样，FuncX也有多个，用于不同的参数个数。

####三. 线程控制
RxJava内置有以下几个线程
- Schedulers.immediate(): 直接在当前线程运行，相当于不指定线程。这是默认的 Scheduler。
- Schedulers.newThread(): 总是启用新线程，并在新线程执行操作。
- Schedulers.io(): I/O 操作（读写文件、读写数据库、网络信息交互等）所使用的 Scheduler。行为模式和 newThread() 差不多，区别在于 io() 的内部实现是是用一个无数量上限的线程池，可以重用空闲的线程，因此多数情况下 io() 比 newThread() 更有效率。不要把计算工作放在 io() 中，可以避免创建不必要的线程。
- Schedulers.computation(): 计算所使用的 Scheduler。这个计算指的是 CPU 密集型计算，即不会被 I/O 等操作限制性能的操作，例如图形的计算。这个 Scheduler 使用的固定的线程池，大小为 CPU 核数。不要把 I/O 操作放在 computation() 中，否则 I/O 操作的等待时间会浪费 CPU。
- 另外， Android 还有一个专用的 AndroidSchedulers.mainThread()，它指定的操作将在 Android 主线程运行。

#####有了这几个 Scheduler ，就可以使用 subscribeOn() 和 observeOn() 两个方法来对线程进行控制了。

- **subscribeOn():** 指定 subscribe() 所发生的线程，即 Observable.OnSubscribe 被激活时所处的线程。或者叫做事件产生的线程。
- **observeOn()**: 指定 Subscriber 所运行在的线程。或者叫做事件消费的线程。

代码示例

    Observable.just(1, 2, 3, 4)
    // 指定 subscribe() 发生在 IO 线程
    .subscribeOn(Schedulers.io()) 
    // 指定 Subscriber 的回调发生在主线程
    .observeOn(AndroidSchedulers.mainThread()) 
    .subscribe(new Action1<Integer>() {
        @Override
        public void call(Integer number) {
            Log.d(tag, "number:" + number);
        }
    });


####四. 转换

1. **map** 将传进来的参数，以另一种对象返回，代码示例
             
             Observable.just("images/logo.png") 
              .map(new Func1<String, Bitmap>() {
	       @Override
	        public Bitmap call(String filePath) { // 参数类型 String
	            return getBitmapFromPath(filePath); // 返回类型 Bitmap
	        } }) 
        .subscribe(new Action1<Bitmap>() {
	        @Override
	        public void call(Bitmap bitmap) { // 参数类型 Bitmap
	            showBitmap(bitmap);
	        } });
2. **FlatMap** ：1. 使用传入的事件对象创建一个 Observable 对象；2. 并不发送这个 Observable, 而是将它激活，于是它开始发送事件；3. 每一个创建出来的 Observable 发送的事件，都被汇入同一个 Observable ，而这个 Observable 负责将这些事件统一交给 Subscriber 的回调方法。这三个步骤，把事件拆成了两级，通过一组新创建的 Observable 将初始的对象『铺平』之后通过统一路径分发了下去。而这个『铺平』就是 flatMap() 所谓的 flat。

