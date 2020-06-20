# a try/catch quiz

在 async/await 出来后，在代码中使用了越来越多的 try/catch, 但是很多时候对它并不了解，以及 finally，接下来看几个小栗子

## When you throw a catch

如果在 catch 里抛出了一个错误，那么 finally 还会运行嘛。看下面的[例子](https://codepen.io/anon/pen/LqMoJd?editors=1111)

```jsx
function example() {
  try {
    throw new Error('出错啦')
  } catch (e) {
    console.log('catch 执行')
    throw e
  } finally {
    console.log('finally 运行')
  }
  console.log('最后执行')
}

example()
```

_catch 和 finally 的 `console.log`都执行了，但是最后一个 console.log 并没有执行_

### Try without catch

当只有 try / finally 的时候[例子](https://codepen.io/anon/pen/LqMoJd?editors=1111)

```jsx
function example1() {
  try {
    console.log('try')
  } finally {
    console.log('finally 运行')
  }
  console.log('最后执行')
}

example1()
```

_尽管没有 catch finally 仍然被执行了。当然即使有 catch finally 还是会被执行_

### Return and finally

如果在 try 里面返回,那么 finally 还会执行嘛[例子](https://codepen.io/anon/pen/LqMoJd?editors=1111)

```jsx
function example2() {
  try {
    console.log('return try')
    return
  } finally {
    console.log('finally 运行')
  }
}

example2()
```

_即使在 try 里进行了 return finally 仍然会执行_

### The Rule

finally 在 try/catch/finally 里，无论如何都会执行。所以可以在 finally 处理 loading 状态。那么`Promise.finally`是否也是一样。

### Promise fulfilled

当 promise 的状态为`fulfilled`的时候, 执行了[finally](https://codepen.io/anon/pen/BMvgdK?editors=1112)

```jsx
function example() {
  Promise.resolve(1)
    .then(() => {
      console.log('resolve')
    })
    .finally(() => {
      console.log('finally')
    })
}

example()
```

### Promise rejected

当 promise 的状态为`rejected`的时候, 是否执行了[finally](https://codepen.io/anon/pen/BMvgdK?editors=1112)

```jsx
function example1() {
  Promise.reject(1)
    .then(() => {
      console.log('resolve')
    })
    .catch(() => {
      console.log('catch')
    })
    .finally(() => {
      console.log('finally')
    })
}

example1()
```

_当 promise 的状态为`rejected`的时候, finally 仍然会被执行。_

这也是本文的灵感所在。[promise 文档](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Promise)  
比如在请求接口的时候，在接口没有返回之前显示 loading 信息，接口返回后无论成功还是失败都应该取消 loading, 来个[栗子 🌰](https://codesandbox.io/s/v10k0xyqwl)

```jsx
import React, { useState, useEffect } from 'react'

export default function LoadingMessage() {
  const [loading, setLoading] = useState(false)

  const getMessage = () => {
    setLoading(true)
    return new Promise(resolve => {
      setTimeout(() => {
        resolve('1')
      }, 3000)
    })
      .then(() => {
        console.log('success')
      })
      .finally(() => {
        setLoading(false)
      })
  }

  useEffect(() => {
    setLoading(true)
    getMessage()
  }, [setLoading])

  return (
    <div>
      <button onClick={() => getMessage()}>点击加载</button>
      {loading ? '加载中...' : 'loading success'}
    </div>
  )
}
```

_对于无论是成功还是拒绝状态都要执行的，就可以放在 finally 里进行处理啦_
