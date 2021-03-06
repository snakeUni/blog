# React.forwardRef

`forwardRef` 主要目的是为了解决高阶组件中无法拿到 ref 的问题，以及在函数式组件中，可以直接拿到原生的 DOM 元素 [forwardRef](https://reactjs.org/docs/react-api.html#reactforwardref)

接下来主要看下 `forwardRef` 的用法，以及在 `hooks` 中的相关用法

## 高阶组件中的 `ref`

在类组件中， 想要调用原生的事件， 我们的做法通常是暴露一个方法， 在方法内部调用原生的方法， 比如 `input` 的 `focus` 事件

```jsx
class Input extends React.Component {
  focus = () => {
    this.inputRef.focus()
  }

  render() {
    return <input ref={el => (this.inputRef = el)} />
  }
}
```

在组件外部， 想要主动使 `input` 聚焦

```jsx
const testRef = React.createRef()

<Input ref={testRef} />

testRef.current.focus()
```

用过传递 `ref`, 调用组件内部的 `focus` 方法， 使 `input` 主动聚焦

那如果此时有一个高阶组件封装了这个 `Input` 组件 如何调用 `focus` 使 input 主动聚焦

```jsx
function Hoc() {
  return <Input />
}

;<Hoc />
```

此时， 如何调用 `Input` 里的 `focus` 方法， 似乎是没有办法(也有可能是我不知道), `forwardRef` 由此而生

```jsx
const testRef = React.createRef()

function Hoc(props, ref) {
  return <Input {...props} ref={ref}/>
}

React.forwardRef(Hoc)

<Hoc ref={testRef}/>

testRef.current.focus()
```

此时 `testRef.current.focus()` 就可以直接调用， 使 `input` 聚焦

## 函数式组件 直接操作 `DOM`

我们不妨把 `Input` 改正函数式组件

```jsx
const Input = (props, ref) => {
  return <input {...props} ref={ref} />
}

export default forwardRef(Input)
```

此时 `Input` 的 `ref` 直接挂在到 `DOM` 上了， 为了使 `input` 聚焦

```jsx
const inputRef = React.createRef()

<Input ref={inputRef}/>

inputRef.current.focus() // 聚焦 😊
inputRef.current.blur() // 失焦 😈
```

## hooks 中使用

如果想要达到和 `class` 相同的效果， 在函数中该如何做, `hooks` 提供了相应的 api

```jsx
function Input(props, ref) {
  const inputRef = React.useRef()
  React.useImperativeHandle(ref, () => ({
    focus: () => {
      inputRef.current.focus()
    },
    blur: () => {
      inputRef.current.blur()
    }
  }))
  return <input ref={inputRef} />
}

export default forwardRef(Input)
```

使用 `Input`

```jsx
const inputRef = React.useRef()

<Input ref={inputRef} />

inputRef.current.focus() // 聚焦 😊
inputRef.current.blur() // 失焦 😈
```

想要详细了解， 前往 [官网](https://reactjs.org/docs/react-api.html#reactforwardref)

`React` 官网团队 有了新的[提议](https://github.com/reactjs/rfcs/pull/107#issuecomment-466304382)可能会解决 `forwardRef` 这种复杂的用法
