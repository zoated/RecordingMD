# TypeScript和Redux的学习与应用
以往TypeScript和Redux的学习都是通过看官方文档做笔记，自己搭建小项目去探索、使TypeScript和Redux的用法。但是当在真正的项目中使用这两个工具时，发现自学的知识储备量的不足以及不熟练，而且实现的方式往往不太优雅且存在心智负担，下面来总结一下我在项目中学习的关于TypeScript和Redux的更加优雅、高级的实现方式。

## TypeScript
以往实现多个相同属性的类型定义
```javascript
interface Animals {
    species: string
    name: string
    color: string
}
// 扩展性更好
interface Fish extends Animals {
    hobby: string
}

interface AnimalsInfo {
    fish: Fish
    bird: Animals
    monkey：Animals
    mouse: Animals
}

```
现在使用Record
```javascript
interface Animals {
    species: string
    name: string
    color: string
} 
type AnimalsInfo = Record<'fish' | 'bird' | 'monkey' | 'mouse', Animals>
```
***

以往实现每个属性都是可选或者仅读
```javascript
interface Animals = {
    species?: string
    name?: string
    color?: string
}

interface Animals = {
   readonly species: string
   readonly name: string
   readonly color: string
}
```

现在使用Partial和readonly
```javascript
type Animals = Partial<{
    species: string
    name: string
    color: string
}>

type Animals = readonly<{
    species: string
    name: string
    color: string
}>
```

***

通过类型断言去添加预期的类型
```javascript
//截取路由中的type,通过类型断言设置预期路由中type的值
getQueryString('type') as 'update' | 'reply' | 'add'
```

***

以往将多个类型合并为一个类型
```javascript
interface Animals1 {
    species1: string
    name1: string
    color1: string
}
interface Animals2 {
    species2: string
    name2: string
    color2: string
}
interface Animals3 {
    species3: string
    name3: string
    color3: string
}
interface AnimalsInfo extends Animals1,Animals2,Animals3
```
现在使用交叉类型
```javascript
type AnimalsInfo = Animals1 & Animals2 & Animals3
```

***
不得不说在使用TypeScript后才发现TypeScript的好,通过类型检测在开发阶段就能发现很多潜在的问题，bug少了，程序莫名崩溃的情况也几乎不存在了，bug排查也因为代码可读性的提高更容易排查了，虽然在开发阶段会增加一些开发成本，但是能够减少我们很多心智的负担，减少维护的成本。

## Redux
#### 使用useSelector、useDispatch替代connect繁琐操作
以往通过connect连接 Redux 和 React
```javascript
// connect方法用于从 UI 组件生成容器组件。connect的意思，就是将这两种组件连起来。
// connect方法接受两个参数：mapStateToProps和mapDispatchToProps。它们定义了 UI 组件的业务逻辑。前者负责输入逻辑，即将state映射到 UI 组件的参数（props），后者负责输出逻辑，即将用户对 UI 组件的操作映射成 Action。
connect(mapStateToProps,mapDispatchToProps)(uiComponent)
```
现在通过useSelector从redux的store对象中提取数据(state)、useDispatch获取dispatch分发action触发redux的store对象中的state变化
```javascript
    const {state} = useSelector(state => state.xxx)

    const dispatch = useDispatch()
    dispatch(actions.xxx(data))
```
***

#### 使用 applyMiddleware() 来增强 createStore()实现异步action
```javascript
createStore(
  reducer,
  undefined,
  applyMiddleware(redux-thunk / redux-promise) //任何被发送到 store 的 action 都会经过thunk,在这里你可以await处理你的异步逻辑，等待异步结果返回后再dispatch分发action
) as Store
```

***



