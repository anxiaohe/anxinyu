## [Rxjs](https://www.bilibili.com/video/BV1qr4y1U7Mp?p=12&share_source=copy_web)

用于异步编程的库。

1. 可观察对象(Observable)：类比Promise，内部可执行异步函数，通过内置的方法将异步执行的结果发送到外部。

2. 观察者(Observer)：类似于then方法中的回调函数，用于接收可观察对象发送的数据。他是一组回调函数的对象，有 next、error、complete

3. 订阅(subseribe)：类似于then方法，用于将可观察对象和观察者联系起来。

   [入门概念理解](https://juejin.cn/post/7003328753556258846#heading-6)

```
// 举个例子 js文件
const _rxjs = require('rxjs');

const { Observable } = _rxjs;
// 定义一个可观察对象
const observable = new Observable(function (obs) {
  setTimeout(function () {
  	// 3秒后将将数据通过观察者的next方法发送给观察者
    obs.next({
      name: '张三'
    });
  }, 3000)
})
// 观察者中的next方法用来接收可观察对象的数据
const observer = {
  next: function (val) {
    console.log(val);
  }
}
// 使用可观察对象的 subscribe 方法将可观察对象和观察者联系起来
observable.subscribe(observer)
```

1. 可观察对象可多次调用 next 方法向外发送数据。

   ```
   const observable = new Observable(function (obs) {
     let index = 0;
     const timer = setInterval(function () {
       obs.next(index++);
     }, 1000)
   })
   const obser = {
     next: function (val) {
       console.log(val);
     }
   }
   observable.subscribe(obser);
   // 如果只是要获取数据，不用传入一个观察者对象，可直接接传入一个函数
   observable.subscribe(value => console.log(value));
   ```

2. 观察者可以使用 complete 方法停止发送数据

   ```
   const observable = new Observable(function (obs) {
     let index = 0;
     const timer = setInterval(function () {
       obs.next(index++);
       if (index === 3) {
       	obs.complete();
       	clearInterval(timer);
       }
     }, 1000)
   })
   const obser = {
     next: function (val) {
       console.log(val);
     },
     complete: function () {
     	console.log('停止')
     }
   }
   observable.subscribe(obser)
   ```

3. 可观者对象内部发生错误时可使用 error 方法来传递错误。当执行了complete方法和error方法后不会再执行next方法。

   ```
   const observable = new Observable(function (obs) {
     let index = 0;
     const timer = setInterval(function () {
       obs.next(index++);
       if (index === 3) {
       	obs.error('报错');
       	clearInterval(timer);
       }
     }, 1000)
   })
   const obser = {
     next: function (val) {
       console.log(val);
     },
     complete: function () {
     	console.log('停止')
     },
     error: function (error) {
     	console.error(error);
     }
   }
   observable.subscribe(obser)
   ```

4. 可观察对象只有在被订阅时才会执行内部方法。可观察对象可被多次订阅，每次订阅都会执行内部方法。

   ```
   const observable = new Observable(function (obs) {
     console.log(11)
   })
   observable.subscribe()
   observable.subscribe()
   observable.subscribe()
   ```

5. 清理 `Observable`执行

   ```
   const observable = new Observable(subscriber => {
       subscriber.next('A');
       setTimeout(() => {
            subscriber.next('B');
       }, 2000);
       setTimeout(() => {
            subscriber.next('C');
       }, 4000);
   })
   const subscription = observable.subscribe(val => console.log(val));
   setTimeout(() => {
       subscription.unscbscribe();
   }, 3000)
   ```
   
6. 在结束订阅之后进行一些操作

   ```
   const observable = new Observable(subscriber => {
       subscriber.next('A');
       setTimeout(() => {
            subscriber.next('B');
       }, 2000);
       setTimeout(() => {
            subscriber.next('C');
            subscriber.complete();
       }, 4000);
       return () => {
       	console.log('Teardown')
       }
   })
   const observal = {
     next: val => console.log(val),
     error: err => console.error(err),
     complete: () => console.log('stop')
   }
   observable.subscribe(observal);
   在取消订阅之后会在Observable回调函数返回的函数中执行一些操作，进行一些取消和清理的方法。
   ```

- **Subject**

  用于创建一个空的可观察对象，可以在外部调用 next 方法。

  ```js
  const subject = new Subject();
  subject.subscribe(obser);
  setTimeout(function () {
  	subject.next('1111')
  }, 1000)
  ```

- **BehaviorSubject**

  和 `Subject` 类似，但可以在创建时传入默认值，一绑定观察者就执行一次`next`方法。

  ```
  const B_subject = new BehaviorSubject('默认值');
  B_subject.subscribe(obser);
  ```

- **ReplaySubject**

  与`Subject`不同，`ReplaySubject`可以在其他观察者订阅时将历史推送数据推送个给刚订阅的观察者。

  ```
  const R_subject = new ReplaySubject();
  R_subject.subscribe({ next (val) { console.log(val) } });
  R_subject.next(1);
  R_subject.next(2);
  R_subject.next(3);
  R_subject.next(4);
  setTimeout(() => {
  	// 三秒后会再输出一次1234
  	R_subject.subscribe({ next (val) { console.log(val) } });
  }, 3000)
  ```

- **AsyncSubject**

  只有在执行了`complete`方法之后才会输出最后一个推送的数据。

  ```
  const A_subject = new AsyncSubject();
  A_subject.subscribe({ next (val) { console.log(val) }, complete () {}  })
  A_subject.next(1);
  A_subject.next(2);
  A_subject.next(3);
  A_subject.next(4);
  A_subject.next(5);
  setTimeout(() => {
  	// 5秒后输出5
  	A_subject.complete()
  }, 5000)
  ```

- **Cold Observable**

  `Cold Observable`表示观察者每次订阅Observable时都是独立的，Observable每次都会独立执行。只有在被`observers`订阅，才会产生值。

  ```
  // 例如，使用http时，每次订阅都会建立一个的http请求。
  import { ajax } from 'rxjs/ajax';
  const $ajax = ajax('...');
$ajax.subscribe(
  	data => {
  		console.log('a', data)
  	}
  )
  $ajax.subscribe(
  	data => {
  		console.log('b', data)
  	}
  )
  ```
  
- **Hot Observable**

  `hot observable`表示多个`Observer`订阅同一个`Observable`，获取到的数据是同一个数据源，获取的数据的差异受订阅时间的影响，在后面订阅的观察者无法获取订阅前的数据。

  ```
  const button = document.querySelector('button');
  const observable = new Observable(function (obser) {
  	button.addEventListener('click', function (event) {
  		obser.next(event);
  	})
  })
  observable.subscribe((val) => {
  	console.log('A', val.x, val.y)
  })
  observable.subscribe((val) => {
  	console.log('B', val.x, val.y)
  })
  // 三秒后绑定的订阅者无法获取到之前发出的数据
  setTimeout(function () {
  	observable.subscribe((val) => {
          console.log('c', val.x, val.y)
      })
  }, 3000)
  ```

  

  