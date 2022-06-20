---
title: 关于React的性能优化
date: 2022-05-19 09:37
tags: [React, 性能优化, 总结]
---

React.memo，阻止渲染，工作中的情况：style更新，会导致不需要style参数的组件也更新


组件匿名函数会造成无端渲染
```html
<Cmp onClick={() => {}} />
```


useCallback也会造成无端渲染的情况


```React

// cmpA

const [state, setState] = useState();

const handleClick = useCallback((res) => {
	setState(res)
	console.log('handleClick')
}, [state]) 

return <CmpB onClick={handleClick} />

// cmpB

return <button onClick={props.onClick(Math.random())} />
```

按理来说，cmpB不需要state，不需要更新，but，state的改变会导致handleClick的改变。handleClick的改变会导致cmpB更新

