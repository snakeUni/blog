# Cache your event listener

在 js 中，创建两个函数是不等的，对象也是这样。来个栗子看下

```jsx
const a = () => {}
const b = () => {}
a === b // false
```

对象也是这样

```jsx
const c = {}
const d = {}
d === c // false
```

那我们这样写 同一个引用就相等了

```jsx
const d = () => {}
const c = d
c === d //true
```

对象也是这样就不多提了，好了回到正题，我们在写 React 的时候经常需要给事件加上监听器,看一下代码

```jsx
import React, { Component } from 'react'

export default class Cache extends Component {
  handleClick = () => {
    alert(1)
  }
  render() {
    return (
      <div>
        <button onClick={() => this.handleClick()}>点击测试</button>
      </div>
    )
  }
}
```

我们'正常（_不正常_）的代码很多都是这样，当然不包含会有人来写`黑科技`啦。在这个 click 函数里，每次点击都会创建一个新的函数,因为这里我们需要传个参数。大家先不要急，传变化的参数后面会说到的。那对于这个我们应该怎么改呢。我们可以这么做？

```jsx
import React, { Component } from 'react'

const handleClick = () => alert(1)

export default class Cache extends Component {
  render() {
    return (
      <div>
        <button onClick={handleClick}>点击测试</button>
      </div>
    )
  }
}
```

或者

```jsx
import React, { Component } from 'react'

const handleClick = () => alert(1)

export default class Cache extends Component {
  handleClick = () => {
    alert(1)
  }
  render() {
    return (
      <div>
        <button onClick={this.handleClick}>点击测试</button>
      </div>
    )
  }
}
```

上面大家都知道，那有列表的情况下呢

```jsx
import React, { Component } from 'react'

const list = [1, 2, 3, 4, 5, 6]

export default class Cache extends Component {
  handleClick = item => {
    alert(item)
  }
  render() {
    return (
      <div>
        {list.map(item => (
          <button onClick={() => this.handleClick(item)}>{item}</button>
        ))}
      </div>
    )
  }
}
```

这个代码是我们正常的写法了，在每次点击的时候都会生成一个新的函数，那这个我们其实就可以利用缓存函数了，这个缓存函数可以自己写比如我们这样改

```jsx
import React, { Component } from 'react'

const list = [1, 2, 3, 4, 5, 6]

export default class Cache extends Component {
  cache = {}

  getCache = key => {
    if (!Object.prototype.hasOwnProperty.call(this.cache, key)) {
      this.cache[key] = () => alert(key)
    }
    return this.cache[key]
  }
  render() {
    return (
      <div>
        {list.map(item => (
          <button onClick={this.getCache(item)}>{item}</button>
        ))}
      </div>
    )
  }
}
```

这样只在第一次的时候会创建一个新的函数，在后面的每一次都会引用之前的函数啦。
