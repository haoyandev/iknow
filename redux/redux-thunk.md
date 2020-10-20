# redux-thunk

> 由于redux中的reducer只接受普通对象的action，即dispatch的只能是一个普通对象，如果想dispatch一个函数，在函数里调用某些api，然后再返回action，就需要redux-thunk结合redux的applyMiddleware一起食用，使用了redux-thunk，createActiontor可以返回函数，并且可以接受dispatch和getState参数

```js
import { applyMiddleware, createStore } from 'redux';
import thunk from 'redux-thunk';
 const store = createStore(
  reducers, 
  applyMiddleware(thunk)
)
 
// store.js
// ...
 const actions = {
   setInfo: () => (dispatch,getState) => {
     // 发送api
     // const {data} = await axios.get(url)
     // dispatch({type: SET_INFO, INFO:data})
   }
 }
// ...
```

