# The Dep Of Use Hook

昨天是 `10月1日` 国庆节，首先祝贺伟大的祖国 70 年生日快乐！！！！

今天是国庆节的第二天，在这里先祝福大家国庆节快乐，吃好喝好！！！

今天闲来无事，不由自由的就开了电脑，这可能是做技术的人的通病？感觉有的时候不自觉就会打开电脑，看些技术
文章呀什么的。正好在看文章的时候，想起来之前在工作中亦是是在学习中遇到一些问题，今天就在这篇文章里总结
一下吧。

## 目录

- 常见 `Hook` 的用法
- `Hook` 的注意项以及规则
- 如何处理 `Hook` 的依赖

### 常见 `Hook` 的用法

React 发布了很多 `Hook`, 今天在这篇文章里主要减少几个 Hook 分别是 `useEffect`, `useMemo`, `useCallback`
为什么只介绍这三个，因为这三个都涉及到依赖项的问题。`useLayoutEffect` 这个不算详细介绍，参考 `useEffect` 即可

#### useEffect

`useEffect` 的用法很简单，这个 Hook 通常是用来处理一些副作用的，比如数据请求。你可以当做 class 中的
`componentDidMount` 和 `componentDidUpdate` 的结合体来使用。以前需要请求数据的时候，我们会把请求数据的
操作放在 `componentDidMount` 中。比如

```js{2-4}
class Text extends React.Component {
  componentDidMount() {
    fetch('').then(result => {})
  }
}
```

但是在 Hook 里就可以这样去写

```js
useEffect(() => {
  fetch('').then(result => {})
}, [])
```

此时需要注意的就是 `依赖项` 的问题，这个例子中依赖项是一个空数组，那么就代表只在 `mount` 的时候执行一次，
下一次不会执行这个 `Hook`。

同理， `useEffect` 也可以取代 `componentDidUpdate`。比如在 class 中使用 `update`

```js
class Text extends React.Component {
  componentDidUpdate(preProps, preState) {
    if (preProps.xx !== this.props.xx) {
      // doing
    }
  }
}
```

在 `Hook` 里就可以这样去写

```js
useEffect(() => {
  // doing
}, [props.xx])
```

只要当 `xx` 发生变化的时候就执行这个 `effect`。用起来还是很简单的。

### `Hook` 的注意项以及规则

在使用 `Hook` 的时候一定要遵循它的[规则](https://reactjs.org/docs/hooks-rules.html?no-cache=1)

Don’t call Hooks from regular JavaScript functions. Instead, you can:

- ✅ Call Hooks from React function components.
- ✅ Call Hooks from custom Hooks (we’ll learn about them on the next page).

必须要遵循上述的规则，才能正确的使用 `Hook`

### 如何处理 `Hook` 的依赖

在使用 `Hook` 的时候，通常会装官方提供的 [hook eslint](https://www.npmjs.com/package/eslint-plugin-react-hooks) 使用
官方提供的 eslint 有很多好处。在一定程度上可以帮助我们避免一些使用 hook 的错误，比如需要在顶层使用。如果你在条件语句中
使用就会报错。使用 lint 减少了错误的发生。但是目前的 eslint 有个问题，这个问题可能会被大家所诟病吧。就是自定添加依赖的问题
当在 eslint 中这样配置

```js
// Your ESLint configuration
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // Checks rules of Hooks
    "react-hooks/exhaustive-deps": "warn" // Checks effect dependencies
  }
}
```

`react-hooks/exhaustive-deps` 这个规则会检查依赖项，常见有 `useEffect`, `useMemo`, `useCallback`
会自动给这个几个 `Hook` 加上依赖，那么这样就会存在问题，有的时候可能我就是不想要这个依赖，但是因为 lint 配置了这个
的原因，自动加上了依赖，遇到这种情况应该怎么去解决或者说如何优雅的来解决这个问题呢？（`删掉这个规则美滋滋-最好不要删`）

这个在 React 的 github [issue](https://github.com/facebook/react/issues/14920) 中有提到过这样的问题，就是希望在使用这个
规则的时候不要自动加上依赖，可以给出警告，但是不要自动添加到代码中。今天从官方的 issue 来分析一个一个这样的例子，是否可以在不去掉这个
规则的前提下，解决这样的依赖问题。

#### Example-1

[codesandbox](https://codesandbox.io/s/nqy69ol00)

This is an uncontrolled Checkbox component which takes a defaultIndeterminate prop to set the indeterminate status on initial render (which can only be done in JS using a ref because there's no indeterminate element attribute). This prop is intended to behave like defaultValue, where its value is used only on initial render.

The rule complains that defaultIndeterminate is missing from the dependency array, but adding it would cause the component to incorrectly overwrite the uncontrolled state if a new value is passed. The dependency array can't be removed entirely because it would cause the indeterminate status to be fully controlled by the prop.

I don't see any way of distinguishing between this and the type of case that the rule is meant to catch, but it would be great if the final rule documentation could include a suggested workaround. 🙂

`原始代码片段`

```js{9-13}
import React from 'react'
import ReactDOM from 'react-dom'

function Checkbox(props) {
  const { defaultIndeterminate, ...rest } = props

  const inputRef = React.useRef(null)

  React.useLayoutEffect(() => {
    if (defaultIndeterminate != null && inputRef.current) {
      inputRef.current.indeterminate = defaultIndeterminate
    }
  }, [])

  return <input {...rest} ref={inputRef} type="checkbox" />
}

const rootElement = document.getElementById('root')
ReactDOM.render(<Checkbox defaultIndeterminate />, rootElement)
```

这个例子会报一个警告就是 `useLayoutEffect` 缺少一个依赖就是 `defaultIndeterminate`, 那么此时这个依赖放在
这里是否合适呢？明显看出是不合适的，因为 `default` 的效果是什么就是初始化或者说只对于 mount 的时候起作用，下一次
修改 default 的值不应该不在应用这个值。但是如果作为依赖，那么每次这个值发生变化都会导致重新应用这个值，如果达到这个作用
不就是 value 的效果了嘛。而不是 `defaultValue` 或 `defaultIndeterminate` 的效果。如果要想达到 default 的效果又不是
disabled lint, 可以试试这样来修改

```js{5}
function Checkbox(props) {
  const { defaultIndeterminate, ...rest } = props

  const inputRef = React.useRef(null)
  const indeterminate = React.useState(defaultIndeterminate)

  React.useLayoutEffect(() => {
    if (indeterminate != null && inputRef.current) {
      inputRef.current.indeterminate = indeterminate
    }
  }, [indeterminate])

  return <input {...rest} ref={inputRef} type="checkbox" />
}
```

使用 `useState` 把 `indeterminate` 作为依赖。

总结： 如果要达到 `default` 的效果其实都可以这么来处理。使用 useState 可以达到缓存的效果

#### Example-2

第二个场景还是很简单的在 issue 中是这样的

```js
const foo = () => {...};
useEffect(() => {
    foo();
  }, []);
```

在外部创建了 `foo` 函数，但是调用需要在 useEffect 内部调用也有可能在其他的事件函数中调用。这种场景在开发中还是很常见的一种。这种场景下
会把 `foo` 加入到依赖中。但是每次当组件 `rerender` 的时候, foo 都是一个新的函数，那么这种场景下如何处理呢？

这种分两种情况来处理，第一就是如果函数在 `effect` 内部使用，并且这个函数并不会在其他地方调用，那么这个函数就需要`声明在 effect 内部`,
第二就是这个函数不仅在 `effect` 内部也需要在其他地方使用。那么此时需要使用 `useCallback` 进行包裹起来，在 `useCallback` 中
处理好依赖关系就可以。第二种方法就是使用 useState, 之前说过 useState 也能起到缓存的作用。那么如果对函数进行缓存，也是可以
达到一样的效果的

#### Example-3

如果在 `useEffect` 内部直接使用 `props.xxx`, 那么此时会自动把 props 加入到依赖中。比如

```js
useEffect(
  function () {
    props.setLoading(true)
  },
  [props.setLoading]
)
```

这不是我们所期望的，所以对于在 useEffect 中使用 props, 最好先解构出相应的属性。 The best practice is always destructuring.

```js{1}
const { setLoading } = props
useEffect(
  function () {
    setLoading(true)
  },
  [setLoading]
)
```

#### Example-4

这是一个常用于数据请求的例子，在 `useEffect` 内部发送请求

```js{21-29}
const useDataApi = url => {
  const [data, setData] = useState('')
  const [isLoading, setIsLoading] = useState(false)
  const [isError, setIsError] = useState(false)

  const fetchData = async () => {
    let response
    setIsError(false)
    setIsLoading(true)

    try {
      response = await fetch(url).then(response => response.json())

      setData(response)
    } catch (error) {
      setIsError(true)
    }

    setIsLoading(false)
  }

  useEffect(
    () => {
      fetchData()
    },
    // react-hooks/exhaustive-deps gives me warning that I should put fetchData into
    // dependency array & remove url
    // if I put fetchData as a dependency and remove url the request will be fired continuously
    [url]
  )

  return { data, isLoading, isError }
}
```

在这个例子中，`useEffect` 是需要 `fetchData` 作为依赖，但是当需要函数作为依赖的时候正如之前所说的，有三种方法可以解决这个。第一把这个函数放在内部，
第二使用 `useCallback`, 第三使用 ref 或者 state 记录该函数。其实还有一种就是把这个函数放在这个 `Hook` 的外部也是一个不错的选择。不妨来一步一步的修改。
首先把函数放在内部应该是什么样的？

```js
useEffect(
  () => {
    const fetchData = async () => {
      let response
      setIsError(false)
      setIsLoading(true)

      try {
        response = await fetch(url).then(response => response.json())

        setData(response)
      } catch (error) {
        setIsError(true)
      }

      setIsLoading(false)
    }

    fetchData()
  },
  // react-hooks/exhaustive-deps gives me warning that I should put fetchData into
  // dependency array & remove url
  // if I put fetchData as a dependency and remove url the request will be fired continuously
  [url]
)
```

![data](./data.jpg)

可以看出此时依赖问题已经解决。其他的都是一样的处理，这个例子就不做过多的阐述。

#### Example-5

第五个例子也是业务中常见的一个例子，就是模拟 `componentDidUpdate` 正如我们所知道的， `useEffect` 有这个作用。但是遇到这种情况

```js
componentDidUpdate(preProps, preState) {
  if (preProps.xxx !== this.props.xxx) {
    fetchData(url)
  }
}
```

例子代码

```js
const MyComponent = props => {
  const { getData, someId, refreshRequest } = props

  useEffect(
    () => {
      const fetchData = () => {
        getData(someId)
      }
      fetchData()
    },
    [refreshRequest] //This is what I want -- just invoke useEffect on intial component mount and when refreshRequest is updated
    //[refreshRequest, someId, getData] //Linter wants everything in useEffect function second array argument, or nothing.  So, if any prop changes, then it will invoke useEffect. Not what I want
  )

  return (
    <div>
      My component has rendered. Check out the console for when data is
      'fetched'
    </div>
  )
}
```

这个在 `useEffect` 中很不好处理，为什么？因为 `fetchData` 和 `url` 都被认为是依赖，实际上真正的依赖应该是 `xxx`

```js
const { xxx, fetchData, url } = props
useEffect(() => {
  fetchData(url)
}, [xxx])
```

这个就会报警告。那么这样的问题如何解决呢？按照之前说的方法，第一我们使用 `useCallback` 来试试看

```js
const fetch = useCallback(() => {
  fetchData(url)
}, [fetchData, url])

useEffect(() => {
  fetch()
}, [fetch])
```

此时 `fetchData` 和 `url` 都会被加到依赖中。这个还是不合理的，因为只要 url 或 fetchData 变化就会触发数据的请求，而与 `xxx` 这个属性毫无关系。这个例子不妨在分析一下
使用了 `props` 但是有一些 props 是依赖于其他的 props, 这种更像是 class 中的 `getDerivedStateFromProps`. 那么不妨按照这个方向来处理这个依赖的问题。使用第二种方法
`useState` 来解决这个问题。

```js{4-7}
const [currentGetData, setCurrentGetData] = useState(getData)
const [prevRefreshRequest, setPrevRefreshRequest] = useState(refreshRequest)

if (prevRefreshRequest !== refreshRequest) {
  setPrevRefreshRequest(refreshRequest)
  setCurrentGetData(getData)
}

useEffect(() => {
  currentGetData(someId)
}, [currentGetData, someId])
```

模拟了 `getDerivedStateFromProps` 来解决这个问题。但是这个例子并不是一个可取的例子，将异步函数传递下来。很有可能会出现 `竞态` 的情况。就是两个请求发出的时候，第一个请求比
第二个请求要慢。实际上我们需要是第二个请求返回的结果。所以最好处理好这样的问题。如果请求数据的就是这个组件的本身的话，可以在 `useEffect` 的 clean 函数中设置`忽略`标志以解决这种
`竞态`的场景。当然如果 `React` 后续的 `Suspense` 出来了，数据请求放在 `Suspense` 中是最好的选择。

## Conclusion

对于使用 `useEffect` 的依赖可以采取四种方式来解决。

- 使用 `useState` 进行缓存。同时可以用来模拟 class 中的 `getDerivedStateFromProps`
- 对于在 `useEffect` 外部的声明，内部又要使用这个函数。可以放入到 `useEffect` 内部
- 对于不能放在 `useEffect` 内部的，可以考虑是否使用 `useCallback` 来解决这样的问题
- 使用 `ref` 或者 放到组件外面

对于这个 [Feedback for 'exhaustive-deps' lint rule](https://github.com/facebook/react/issues/14920) Dan 也在里面给出了一些解决办法。可以参考
