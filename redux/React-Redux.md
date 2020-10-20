# React-Redux

## redux的缺点

1. 组件如果需要用到store里面的状态，都要各自引入store对象或者是把store挂在window对象中
2. 如果想在状态更新后组件重新渲染展示最新的状态值，需要用到store.subscribe()挂在监听函数，重新渲染组件或者执行setState



## redux-redux

**把store直接集成到React应用的顶层props里面，只要各个子组件能访问到顶层props就行， 并且提供Provide和connect函数，本质上是利用react中的context机制**

1. Provide

   - Provider包裹在根组件最外层，使所有的子组件都可以拿到state

   - Provider接收store作为props，然后通过context往下传递，这样react中任何组件都可以context获取到store

   - ```jsx
     import React from 'react'
     import ReactDOM from 'react-dom'
     import {Provider} from 'react-redux'
     
     import App from './App'
     import store from './store'
     
     ReactDOM.render(
       <Provider store={store}>
         <App />
       </Provider>,
       document.getElementById('root')
     )
     ```

     

2. connect

   - Provider内部组件如果想要使用到state中的数据，就必须要connect进行一层包裹封装，换句话说就是必须要被connect进行加强

   - 本质上是在组件外部再包裹一层，当状态值发生改变后会把状态值当作组件的props传入，一旦props改变就会重新渲染组件，节省程序员手动更新UI的步骤

   - 可以接受两个函数参数，分别是mapStateToProps和mapDispatchToProps

     - mapStateToProps接受两个参数(state, ownProps)返回一个对象，该对象会成为属性作为组件props的一部分，如果值为null，组件不会监听store的变化，也就是说Store的更新不会引起UI的更新
     - mapDispatchToProps接受两个参数(dispatch, ownProps)返回一个对象，该对象会成为方法成为组件props的一部分，值可以为null

   - ```jsx
     import react from 'react'
     import {connect} from 'react-redux'
     
     function SonComponent (props) {
       	const {count, addCount} = props
     }
     
     function mapStateToProps (state) {
       count: state.count
     }
     
     function mapDispatchToProps (dispatch) {
       addCount: dispatch({type: 'ADD'})
     }
     
     export default connect(mapStateToProps, mapDispatchToProps)(SonComponent)
     ```