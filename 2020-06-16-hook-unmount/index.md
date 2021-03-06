# Unmount Hook

突然发现好久没写文章了，正好这次在项目中遇到了一个文章就写了这篇文章。

在写一个组件的时候，经常需要用一些常用的 Hook, 比如 `useState`, `useRef`, `useEffect` 等。但是有时候也要需要用组件的 unmount,
以前在写 class 组件如果需要用 unmount, 通常会使用 `componentWillUnmount` 声明周期。

```jsx{5-7}
import * as React from 'react'
import ReactDOM from 'react-dom'

class App extends React.Component {
  componentWillUnmount() {
    console.log('unmount')
  }

  render() {
    return <div>demo</div>
  }
}

ReactDOM.render(<App />, document.getElementById('root'))
```

当 App unmount 的时候 componentWillUnmount 将被调用。那么如何使用 `hooks` 来代替 unmount 呢

[官网](https://reactjs.org/docs/hooks-faq.html)上的说法是使用 `useEffect` 来代替, 不妨改造一下上面的代码

```jsx{5-9}
import * as React from 'react'
import ReactDOM from 'react-dom'

function App() {
  React.useEffect(() => {
    return () => {
      console.log('unmount')
    }
  }, [])

  return <div>demo</div>
}

ReactDOM.render(<App />, document.getElementById('root'))
```

可以看出使用 hooks 来改造是简单了很多很多，那么是否 useEffect 能完全解决 componentWillUnmount 的问题呢？[看这样的例子](https://codesandbox.io/s/inspiring-williams-8qerf)

```jsx
import * as React from 'react'

function Demo({ name }) {
  const keyMapRef = useRef(new Map())
  React.useEffect(() => {
    keyMapRef.current.set(name, '100')

    return () => {
      keyMapRef.current.delete(name)
    }
  }, [name])
  return <div>demo</div>
}
```

在这个例子中，每当 name 发生改变，keyMap 就会删除之前的 name 值，然后重新注册一个新的值进去。所以当这样使用 Demo 的时候， name 为空在 mount 会被注册进去，name
更新后，空值的 name 会被删除，重新注册新的 name 即 test

```jsx
import * as React from 'react'
import ReactDOM from 'react-dom'

function App() {
  const [name, setName] = React.useState('')

  React.useEffect(() => {
    setName('test')
  }, [])

  return (
    <div>
      <Demo name={name} />
    </div>
  )
}
```

如果此时突然有这样的一个需求，就是希望在传入某个 props 的时候，希望组件在 unmount 的时候并不会删除注册的 name。假设这个 props 为 keepName, 此时应该怎么做？

正常情况下，我们会这么修改 Demo 组件

```jsx
import * as React from 'react'

function Demo({ name, keepName }) {
  const keyMapRef = useRef(new Map())
  React.useEffect(() => {
    keyMapRef.current.set(name, '100')

    return () => {
      if (keepName) return
      keyMapRef.current.delete(name)
    }
  }, [name])
  return <div>demo</div>
}
```

这个代码第一眼看起来是没有问题，第二眼看起来也没有问题对吧。如果使用者传递下来的 name 是一个静态的值，那么这个例子是不存在问题。

```jsx
<Demo name="test" keepName />
```

因为 name 是一个常量字符串，因此代码正常运行。看起来是很完美的，那么如果 name 是动态的呢？因为 name 在不断的改变，每次改变都没有
删除之前的 name, 那么 keyMapRef 里就会不断的增加新的 name。这样明显是有 bug 的。

```jsx{14-19}
import * as React from 'react'
import ReactDOM from 'react-dom'

function App() {
  const [name, setName] = React.useState('')

  React.useEffect(() => {
    setName('test')
  }, [])

  return (
    <div>
      <Demo name={name} keepName />
      <button
        onClick={() => {
          setName(Math.random())
        }}
      >
        setName
      </button>
    </div>
  )
}
```

当点击 setName 时，新的 name 就会被不断的注册进去。很显然 bug 出现了。那么如何只在 unmount 的时候保留 name 呢？因为即使增加一个空依赖的 useEffect 也是不行的

```jsx{12-18}
import * as React from 'react'

function Demo({ name, keepName }) {
  const keyMapRef = useRef(new Map())
  React.useEffect(() => {
    keyMapRef.current.set(name, '100')

    return () => {
      keyMapRef.current.delete(name)
    }
  }, [name])

  React.useEffect(() => {
    return () => {
      if (keepName) return
      keyMapRef.current.delete(name)
    }
  }, [])

  return <div>demo</div>
}
```

即使增加了一个空依赖的 useEffect, 但是当 Demo unmount 的时候，上一个 useEffect 的 return 仍然是会执行的。仍然会把 name 给删除掉。可能又有同学会这样想，
那在 unmount 后在重新 set 一次不就好了嘛

```jsx{16}
import * as React from 'react'

function Demo({ name, keepName }) {
  const keyMapRef = useRef(new Map())
  React.useEffect(() => {
    keyMapRef.current.set(name, '100')

    return () => {
      keyMapRef.current.delete(name)
    }
  }, [name])

  React.useEffect(() => {
    return () => {
      if (keepName) {
        keyMapRef.current.set(name, '100')
      }
    }
  }, [])

  return <div>demo</div>
}
```

此时这个代码还是有 bug 的，因为 `keyMapRef.current.set(name, '100')` 这里的 name 一直都是第一次传递进来的 name, 初始化传进来是空字符串，那么这里的 name 就是
空的字符串。那么又有同学提到，那我时刻让 name 保持最新不就好了嘛(这也是我一开始的做法)

```jsx{7,6,20}
import * as React from 'react'

function Demo({ name, keepName }) {
  const keyMapRef = useRef(new Map())

  const nameRef = useRef(name)
  nameRef.current = name

  React.useEffect(() => {
    keyMapRef.current.set(name, '100')

    return () => {
      keyMapRef.current.delete(name)
    }
  }, [name])

  React.useEffect(() => {
    return () => {
      if (keepName) {
        keyMapRef.current.set(nameRef.current, '100')
      }
    }
  }, [])

  return <div>demo</div>
}
```

用 nameRef 记录最新的 name 在 unmount 的时候使用最新的 name 重新 set 进去就可以保留了之前的 key。这样的做法是没有问题，这也是我一开始的做法，那么如果保留的
不只是有 name 呢？如果后面还有 age, address 是不是所有的都要来记录呢？是否有一个通用的方法来解决呢？看这样的一个例子

```jsx
import * as React from 'react'

function CountDemo() {
  const [count, setCount] = React.useState(0)

  React.useEffect(() => {
    console.log('update count 😁', count)

    return () => {
      console.log('unupdate count 😹', count)
    }
  }, [count])

  return (
    <div>
      <h1>count: {count}</h1>
      <button
        onClick={() => {
          setCount(count + 1)
        }}
      >
        setCount
      </button>
    </div>
  )
}
```

使用 `CountDemo`

```jsx
import * as React from 'react'
import ReactDOM from 'react-dom'

function App() {
  const [show, setShow] = React.useState(true)

  return (
    <div>
      {show && <CountDemo />}
      <button
        onClick={() => {
          setShow(!show)
        }}
      >
        setShow
      </button>
    </div>
  )
}

ReactDOM.render(<App />, document.getElementById('root'))
```

当点击 `setCount` 时，console 的答应顺序是

```
unupdate count 😹 0
update count 😁 1

unupdate count 😹 1
update count 😁 2

unupdate count 😹 2
update count 😁 3
```

可以看出每次都是先 `unupdate` 并且此时拿到的 count 是旧的值。此时点击了 `setCount` 三次，页面上展示的 count 为 3，此时在点击 `setShow`

可以看出这次的打印结果是

```
unupdate count 😹 3
```

因为此时 CountDemo 被 unmount 了，执行了 effect 的 return, 所以此时的 return 拿到的 count 就是目前最新的 count 为 3, 既然知道了这个顺序
那么是否可以改造一下之前的代码呢？

```jsx{13-15}
import * as React from 'react'

function Demo({ name, keepName }) {
  const keyMapRef = useRef(new Map())

  const nameRef = useRef(name)
  nameRef.current = name

  React.useEffect(() => {
    keyMapRef.current.set(name, '100')

    return () => {
      if (nameRef.current === name) {
        return
      }

      keyMapRef.current.delete(name)
    }
  }, [name])

  return <div>demo</div>
}
```

此时增加了一个条件语句

```jsx
if (nameRef.current === name) {
  return
}
```

根据上面得出的结论，那么如果 name 发生了变化执行的 effect 的 return, 那么 `nameRef.current !== name`, 如果是组件在 unmount 的时候执行的 return,
那么此时 `nameRef.current === name`, 此时函数直接返回,无法执行到 delete 语句。

目前只有一个依赖，我们可以这么做，那么如果依赖很多怎么办？那就很难搞了，总不能记录所有的依赖项吧(虽然说是可以的)。此时可以用 React 自带的一些 hook 来解决这个问题，
这里使用 useMemo 来解决这个问题。

```jsx{6-8}
import * as React from 'react'

function Demo({ name, keepName }) {
  const keyMapRef = useRef(new Map())

  const depValue = useMemo(() => ({}), [name])
  const depRef = useRef(depValue)
  depRef.current = depValue

  React.useEffect(() => {
    keyMapRef.current.set(name, '100')

    return () => {
      if (depRef.current === depValue) {
        return
      }

      keyMapRef.current.delete(name)
    }
  }, [name])

  return <div>demo</div>
}
```

此时使用了 useMemo 来记录所有的依赖，这样即使后面增加再多的依赖也不会存在问题了。如果有需求可以封装成一个自动的 hook

```jsx
function useUnmount(dep, fn) {
  const depValue = useMemo(() => ({}), dep)
  const depRef = useRef(depValue)

  useEffect(() => {
    return () => {
      fn(depRef.current === depValue)
    }
  }, [dep])
}
```
