# 前端开发小技巧

- 某些数据可以由其他数据通过计算推断出来，则不需要把它定义为state

  ```
  比如A可由B推断出来
  cosnt A = B && B.操作
  另外作为一个数据，希望它能够尽量保持原貌（数组），因为有可能其他地方需要用到，所以只有在需要的时候再做变形
  ```

- `try` 的部分越少越好，仅在必要的地方 `try` 

- 如果业务逻辑依赖某个值，则在effect最开始判断一下

  ```
  useEffect(() => {
  	if (!A) return
  })
  ```

- Object.assign(fileList, refreshData)

  - `Object.assign` 会修改第一个参数，不符合 immutability 原则，可以直接用 [spread](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_syntax#Spread_in_object_literals)

    ```
    { ...fileList, ...refreshData }
    ```

- 和接口有关的转化越远离业务代码、越靠近接口调用越好

- Interface 命名用字母**I**开头，使用大驼峰规则

- 返回类型要明确，因为组件中要用到

- 虽然接口要的是一个 JSON string，但是 service 封装时暴露的参数最好还是别让应用层 stringify 了，而是要原始的数据，然后在发请求的 service 里 stringify

  

- try的是axios，但抛的错不一定是AxiosError

  ```
  export function isAxiosError(e: Error): e is AxiosError {
    return Boolean((e as AxiosError).config)
  }
  
  apiRequest.interceptors.response.use(undefined, (e: unknown) => {
    if (isAxiosError(e) && e.response?.data?.msg_cn) {
      e.message = e.response.data.msg;
    }
    return Promise.reject(e);
  });
  ```

  