# Promise

## Promise是什么？

- 抽象表达
  promise是JS中进行异步编程的新的解决方案
  旧的方案：纯回调，会形成回调地狱
- 具体表达
  1. 从语法上来说： Promise是一个构造函数
  2. 从功能上来说： promise对象用来封装一个异步操作并可以获取其结果



##  Promise的状态改变

- pending变为resolved

- pending变为rejected

  *** 说明： 只有这两种，且一个promise对象只能变一次
  					无论变为成功还是失败，都会有一个结果数据
  					成功的结果数据一般称为value，失败的结果数据一般称为reason***

  ***

  

## 为什么使用Promise

1. 指定回调函数的方式更灵活
   旧的：必须在启动异步任务前指定
   promise：启动异步任务 => 返回promise对象 => 给promise对象绑定回调函数(甚至在异步任务启动后)
2. 支持链式调用，可以解决回调地狱的问题
   - 什么是回调地狱，回调函数嵌套调用，外部回调函数异步执行结果是嵌套的回调函数执行的条件
   - 回调地狱的缺点？不便于阅读和/不便于异常处理
   - 解决方案？promise的链式调用
   - 终极解决方案？async/await



## 如何使用promise

***API***

1. Promise构造函数：Promise(excutor)
   (1) excutor函数：执行器(resolve, reject) => { }
   (2) resolve函数：内部定义成功时我们调用的函数 value => { }
   (3) reject函数： 内部定义失败时我们调用的函数 reason => { }
   ***说明：***
   ***excutor会在promise内部立即同步回调，异步操作在执行器中执行***

2.  Promise.prototype.then方法：(onResolved, onRejected) => { }
   onResolved函数：成功的回调函数 value => { }
   onRejected函数： 失败的回调函数 reason => { }
   ***说明：***
   ***指定用于得到成功value的成功回调和用于得到失败reason的失败回调函数 返回一个Promise对象***

3. Promise.prototype.catch方法：(onRejected) => { }
   onRejected函数：失败的回调函数 reason => { }
   ***说明***

   ***then()的语法糖,相当于：then(undefined, onRejected

4. Promise.resolve方法：value => { }
   value：成功的数据或Promise对象
   ***说明***
   ***返回一个成功/失败的promise对象***

5. Promise.reject方法：reason => { }
   reason：说明的方法
   ***说明***
   ***返回一个失败的Promise对象***



## Promise的几个关键问题

1. 改变promise状态和指定回调函数谁先谁后？

   - 都有可能，正常情况下是先指定回调函数再改变状态，但也可以先改状态后指定回调函数

   - 如何先改变状态再指定回调函数？

     - 在执行器中直接调用resolve()/reject() 
     - 延迟更长的时间才调用then()

   - 什么时候才能得到数据？

     - 如果先指定的回调，那当状态发生改变时，回调函数就会执行，得到数据
     - 如果先改变的状态，那当指定回调函数时，回调函数就会执行，得到数据

   - promise.then()返回的的新promise的结果状态由什么决定？

     - 简单表达：
       由then()指定的回调函数执行的结果决定

     - 详细表达：

       1）如果抛出异常，新的promise状态变为rejected，reason为抛出异常

       2）如果返回非promise的任意值，新promise值变为resolved，value为返回的结果

       3）如果返回的是另一个新promise，此promise的结果就会变成新的promise的结果



## 手写promise

```js
class Promise {
  // 定义三个状态属性
  static PENDING = 'pending'
  static RESOLVED = 'resolved'
  static REJECTED = 'rejected'

	static resolve (value) {
    // 返回一个新的Promise对象
    return new Promise((resolve, reject) => {
      // 判断value是否为Promise对象
      if (value instanceof Promise) {
        value.then(resolve, reject)
      } else {
        resolve(value)
      }
    })
  }

	static reject (reason) {
    // 返回一个新的Promise对象
    return new Promise((resolve, reject) => {
      reject(reason)
    })
  }

	static all (promises) {
		// 定义一个计数器
    let count = 0
    // 定义一个数组保存结果
    let result = new Array(promises.length)
    // 返回一个新的Promise对象
    return new Promise((resolve, reject) => {
      // 遍历promises
      promises.forEach((p, index) => {
        Promise.resolve(p).then(value => {
          count++
          result[index] = value
          if (count === promises.length) {
            resolve(result)
          }
        }, reason => reject(reason))
      })
    })
  }

	static race (promises) {
    // 返回一个新的Promise对象
    return new Promise((resolve, reject) => {
      // 遍历promises
      promises.forEach(p => {
        Promise.resolve(p).then(value => resolve(value), reason => reject(reason))
      })
    })
  }

	// 定义构造函数
	constructor (excutor) {
    // 定义promise实例的属性
    // 状态默认为pending
    this.status = Promise.PENDING
		// 创建用来保存数据的data属性
    this.data = undefined
    // 创建用来保存回调函数的callbacks数组
    this.callbacks = []
    
    // resolve方法
    this.resolve = value => {
      // 只有状态为pending才可以改变状态
      // 判断当前状态
      if (this.status === Promise.PENDING) {
        // 把状态改为resolved
        this.status = Promise.RESOLVED
        // 保存数据
        this.data = value
        // 遍历callbacks异步执行onResolved函数
        if (this.callbacks.length > 0) {
          this.callbacks.forEach(callbacksObj => {
            // 异步调用
            setTimeout(() => {
              callbacksObj.onResolved(this.data)
            })
          })
        }
      }  
    }
    
    // reject方法
    this.reject = reason => {
      // 只有状态为pending才可以改变状态
      // 判断当前状态
      if (this.status === Promise.PENDING) {
        // 把状态改为rejected
        this.status = Promise.REJECTED
        // 保存数据
        this.data = reason
        // 遍历callbacks异步执行onRejected函数
        this.callbacks.forEach(callbacksObj => {
          // 异步调用
          setTimeout(() => {
            callbacksObj.onRejected(this.data)
          })
        })
      }  
    }
    // 自动同步执行executor
		// 判断excutor是否为函数
		if (typeof excutor !== 'function') {
      throw new TypeError(`Promise resolver ${excutor} is not a function`)
    }
		// 捕获异常
    try {
			excutor(this.resolve, this.reject)
    } catch (error) {
      this.reject(error)
    }

		// then方法
		this.then = (onResolved, onRejected) => {
      // 判断onResolved/onRejected类型
      onResolved = typeof onResolved === 'function'? onResolved: value => value
      // 指定默认的失败回调函数(实现错误/异常穿透的关键点)
      onRejected = typeof onRejected === 'function'? onRejected: reason => { throw reason }
      // 返回一个新的Promise对象
      return new Promise((resolve, reject) => {
        // 定义handle方法用来执行onResolved/onRejected函数 并且改变状态
        let handle = callback => {
          // 执行callback函数
         	// 捕获异常
          try {
            const result = callback(this.data)
            // 判断result是否为Promise类型
            if (result instanceof Promise) {
              result.then(resolve, reject)
            } else {
              resolve(result)
            }
          } catch (error) {
            reject(error)
          }
        }
        // 判断当前状态
        if (this.status === Promise.RESOLVED) {
          setTimeout(() => {
						handle(onResolved)
          })
        } else if (this.status === Promise.REJECTED) {
          setTimeout(() => {
            handle(onRejected)
          })
        } else {
					// 挂载onResolved/onRejected函数
          this.callbacks.push({
            onResolved () {
              handle(onResolved)
            },
            onRejected () {
              handle(onRejected)
            }
          })
        }
      })     
    }

    // cathc方法
    this.catch = onRejected => {
      this.then(undefined, onRejected)
    }
  }
}
```

