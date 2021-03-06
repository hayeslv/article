# 按照以下要求，实现 createFlow 函数

```js
const delay = (ms) => new Promise(resolve => setTimout(resolve, ms));
const subFlow = createFlow([() => delay(1000).then(() => console.log('c'))]);
createFlow([
  () => console.log('a'),
  () => console.log('b'),
  subFlow,
  [() => delay(1000).then(() => console.log('d'), () => console.log('e'))]
]).run(() => {
  console.log('done')
});
// 需要按照 a,b,延迟1秒,c,延迟1秒,d,e,done 的顺序打印
```

按照上面的测试用例，实现 `createFlow`：

- `flow`是指一系列 `effects` 组成的逻辑片段。
- `flow` 支持嵌套。
- `effects` 的执行只需要支持串行。



**题解：**

在本题中，`createFlow` 以一个数组作为参数，数组参数可有以下几种类型：

- 普通函数

```js
() => console.log('a')
```

- 异步函数

```js
() => delay(1000).then(() => console.log('c'))
```

- 嵌套 `createFlow`

```js
const subFlow = createFlow([() => delay(1000).then(() => console.log('c'))])
```

- 数组

```js
[() => delay(1000).then(() => console.log('d'), () => console.log('e'))]
```



**步骤1：扁平化**

因为`effects`的执行只需要支持串行，所以我们可以把数组扁平化一下

```js
function createFlow(effects = []) {
  // 浅拷贝一下，尽量不影响传入的参数
  const queue = [...effects.flat()]
}
```

**步骤2：run执行**

`createFlow`并不是直接执行，而是 `.run()` 之后才会开始执行

```js
function createFlow(effects = []) {
  const queue = [...effects.flat()]
  const run = async function(cb) {
    for(let task of queue) {
      
    }
    if(cb) cb()
  }
}
```

**步骤3：异步函数**

因为参数中有异步函数，这里使用 `await` 执行

```js
function createFlow(effects = []) {
  const queue = [...effects.flat()]
  const run = async function(cb) {
    for(let task of queue) {
      await task()
    }
    if(cb) cb()
  }
}
```

**步骤4：支持嵌套**

使用`isFlow`来判断当前是否嵌套执行

```js
function createFlow(effects = []) {
  const queue = [...effects.flat()]
  const run = async function(cb) {
    for(let task of queue) {
      if(task.isFlow) { // 如果是嵌套，执行嵌套函数
        await task.run()
      } else {
        await task()
      }
    }
    if(cb) cb()
  }
  return {
    run,
    isFlow: true
  }
}
```

其实可以使用 `class` 来实现，用`instanceof`来判断是否是嵌套，代码实现如下：

```js
class Flow() {
  constructor(props) {
    this.queue = props
  }
  async run(cb) {
    for(let task of this.queue) {
      if(task instanceof Flow) {
        await task.run()
      } else {
        await task()
      }
    }
    if(cb) cb()
  }
}
function createFlow(effects = []) {
  return new Flow(effects)
}
```

**完整代码**

- function

```js
function createFlow(effects = []) {
  const qeueue = [...effects.flat()]
  const run = async function(cb) {
    for(let task of queue) {
      if(task.isFlow) {
        await task.run()
      } else {
        await task()
      }
    }
    if(cb) cb()
  }
  return {
    run,
    isFlow: true
  }
}
```

- class

```js
class Flow {
  constructor(effects) {
    this.queue = [...effects.flat()]
  }
  async run(cb) {
    for(let task of this.queue) {
      if(task instanceof Flow) {
        await task.run()
      } else {
        await task()
      }
    }
    if(cb) cb()
  }
};

function createFlow(effects = []) {
    return new Flow(effects)
};
```































