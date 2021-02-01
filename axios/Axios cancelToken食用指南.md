# Axios cancelToken食用指南

1. 先介绍一下需求背景，有一个输入框，会根据用户的输入内容然后发送请求，在做了防抖处理的基础上(200ms)，输入一个较为复杂的内容， 在200ms后，然后立马清空，此时总共有两个请求发出去，一个请求的查询条件较为复杂暂且叫requestA，一个请求的查询条件较为简单(为空)暂且叫requestB，发送顺序是 requestA 先发requestB后发，理想情况页面应该展示的是requestB获取到的数据， 但是由于requestB的查询条件比较简单，接口返回的比较快，就先渲染requestB的数据，然后此时requestA的数据返回了，跳用相应的callback，渲染requestA的数据，导致用户看到的是第一次请求的数据。总结：虽然发送请求有先后顺序，但是返回的时候受到一些网络原因所以返回的顺序不能确定，需要实现后发送的请求取消前面发送的请求。

## 在vue中使用axios cancelToken

这里介绍一种常用的使用方法

```vue
import axios from 'axios'

data () {
	cancelToken: null
},
methods: {
	getData () {
		// 清除上一个请求
		if (this.cancelToken) {
			this.cancelToken.cancel()
		}
		this.cancelToken = axios.cancelToken.source()
		axios.get('/test', {
				cancelToken: this.cancelToken.token
		})
	}
}
```

