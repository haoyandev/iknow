# redux

## 1. state-状态

**就是我们传递的数据，那么我们在用React开发项目的时候，大致可以把State分为三类**

- Domain State：可以理解为服务端的数据，比如：获取用户的信息，商品的列表等等
- UI State：决定当前UI展示的状态，比如： 弹窗的显示隐藏，受控组件等等
- App State：App级别的状态，比如：当前是否请求loading，当前路由信息等可能被多个和组件去使用到的状态

## 2. Action-事件

**Action是把数据从应用传到store的载体，它是store数据的唯一来源，一般来说，我们可以用过store.dispatch()将action传递给store** 

- Action特点：

  1. Action本身就是一个js普通对象
  2. Action对象内部必须要有一个type属性来表示要执行的动作
  3. 多数情况下，这个type会被定义成字符串常量
  4. 除了type字段之外，action的结构随意进行定义
  5. 而我们在项目中，更多的喜欢用action创建函数(就是创建action的地方)
  6. 只是描述了有事情要发生，并没有描述如何去更新state

- ```js
  // action
  {
    type: 'action_name',
    info: {}
  }
  // action创建函数
  function addAction(params) {
    return {
      type: 'add',
      ...params
    }
  }
  ```

## 3.Reducer

**Reducer本质就是一个函数，他用来响应发过来的action，然后经过处理，把state发送给store**

注意： 在Reducer函数中，需要return返回值，这样store才能接收到数据，函数会接受两个参数，第一个参数是初始化state，第二个参数是action



## 4. Store

**sotre就是把action与reducer联系到一起的对象**

- 主要职责：

  - 维持应用的state
  - 提供getState()方法获取state
  - 提供dispatch()方法发送action
  - 通过subscribe()来注册监听
  - 通过subscribe()返回值来注销监听

- ```js
  import { createStore } from 'redux'
  const store = createStore(reducer) 
  ```

  

