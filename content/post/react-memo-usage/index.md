---
title: "React memo usage"
date: 2023-08-11T09:06+08:00
draft: true
description: "React memo usage"
categories:

tags:
    - react
---

# **什么是 React.memo()？**

`React.memo()` 随 React v16.6 一起发布。虽然类组件已经允许您使用 PureComponent 或 shouldComponentUpdate 来控制重新渲染，但 React 16.6 引入了对函数组件执行相同操作的能力。

`React.memo()` 是一个高阶组件 (HOC)，它接收一个组件A作为参数并返回一个组件B，如果组件B的 props（或其中的值）没有改变，则组件 B 会阻止组件 A 重新渲染 。

```js
export const Demo = () => {  
const [time, setTime] = useState<number>(0)  
	return (  
	<>  
		<div>demo</div>  
		<button onClick={() => setTime(time + 1)}>Plus</button>  
		<UseMemoComponent/>  
	</>  
	)  
}  
  
const Component = () => {  
const time = new Date().toTimeString()  
return <div>{time}</div>  
}  
const UseMemoComponent = React.memo(Component)
```
这里使用React.memo包裹的组件在点击父组件的button的时候是不会变化的，一直是最初的那个time。


**什么是 useMemo()？**

`React.memo()` 是一个 HOC，而 `**useMemo()**` 是一个 React Hook。使用 `useMemo()`，我们可以返回记忆值来避免函数的依赖项没有改变的情况下重新渲染。

为了在我们的代码中使用 `useMemo()`，React 开发者有一些建议给我们：

- **您可以依赖** `**useMemo()**` **作为性能优化，而不是语义保证**
    
- **函数内部引用的每个值也应该出现在依赖项数组中**

```js

```

[Reference https://mp.weixin.qq.com/s/zxT2GfujdbQfvrCtRxkbiQ](https://mp.weixin.qq.com/s/zxT2GfujdbQfvrCtRxkbiQ)