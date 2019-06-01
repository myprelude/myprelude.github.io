---
layout: post
title:  "快速上手react Hook开发"
date:   2019-06-01 12:00:23 +0700
categories: [javascript]
---
>作者：myprelude@github  
原文链接： https://myprelude.github.io       
转载请注明出处，保留原文链接和作者信息。

## 为什么react hook

React Hooks 是 React16.7.0-alpha版本推出的新特性。在react16.8.0中正式推出。在我们抱怨学不动的时候是否考虑过为啥需要会产生hook？在我们之前的版本中我们创建组件的方法主要有两种：

`1.通过继承React.Component创建类组件` 

{% highlight javascript %}
export default class App extends React.Component{
    constructor(props){
        super(props)
        this.state = {
            num:0
        }
        this.clickButton = this.clickButton.bind(this)
    }
    clickButton(){
        this.setState({
            num:this.state.num+1
        })
    }
    componentDidMount(){
        console.log('组件已经加载成功+++++++')
    }
    render(){
        return(
            <div>
                <button onClick={this.clickButton}>点我呀{this.state.num}</button>
            </div>
        )
    }
    componentwillUnmount(){
        console.log('组件即将卸载------')
    }
}
{% endhighlight %}
ok！我们创建了一个App的组件，这个组件点击按钮后数字会自动累加，在组件加载和卸载过程中我们打印了日志。现在哪里需要组件直接引入就好了。

`2.通过函数return<JSX>生成函数组件`

{% highlight javascript %}

export default App = (props)=>{
    return(
        <div>
            <button onClick={props.clickButton}>点我呀{props.num}</button>
        </div>
    )
}

{% endhighlight %}

同样我们也以函数的方式创建了一个App组件。

**函数组件**

但是和类组件相比会发现，对于组件的操作（点击）都只能通过props来控制；但是组件的挂载、卸载等生命周期的函数回调我并不能拿到(如日志打印)，所以一般我们函数组件都是用来渲染一些无状态，不需要利用生命周期回调的UI组件的渲染。当组件涉及到很多状态的变更以及在不同的生命走起中需要做不同操作时(例如 接口请求，销毁副作用等)，那么不得不使用类组件。

**类组件**

类组件其实也不是那么完美，比如我们还有 Home Detail等组件也需要打印组件的日志，我们怎么做呢？ctrl+c ctrl+v，明显不合理。当然官方对于这样的场景提供了HOC(高阶组件)、render Props(属性渲染)解决方案，如果还不了解可以查看之前的博文[react 如何实现功能的复用]()。类组件给我们的感觉所有组件都是继承于React.Component，我们是不是可以自己去封装一个MyComponent组件来继承呢？官网页面明确说了，我们主要通过React.Component拓展组件功能而不是继承。在使用类组件时，是不是经常会遇到`undefined is not function`的报错呢？在编写类组件时我们不得不使用箭头函数或者bind(this),来确保this的指向。

**react Hook**

在开发时我们往往都是将一些常用的功能函数封装起来，以便需要时候调用。那么我们编写react 组件的时候能不能也这样去做呢？通过上面分析类组件虽然可以实现功能复用但是和生命周期绑定在一起，增加代码的耦合性显然不是理想的方式。函数组件好像是可以但是拿不到生命周期同时无法直接操作状态显然并不能满足要求。这时Hook诞生了，Hook为函数组件提供法改变状态，操作生命周期的能力。我们可以将组件写成一个个函数，将公共的地方抽出来做成一个自定义Hook以便我们功能复用。

## 开始学习react hook

*(确保package.json里面react版本在16.8以上,没有请升级否则代码无法跑起来)*

将上面的App组件用hook改写下：

{% highlight javascript %}

import React,{useState,useEffect} from react;
function App(){
    const [num,setNum] = useState(0);    // 1
    useEffect(()=>{         // 2
        console.log('组件已经加载成功+++++++');
    })

    return(
        <div>
            <button onClick={()=>{setNum(num+1)}}>点我呀{num}</button>
        </div>
    )
}

{% endhighlight %}

和上面的函数组件是不是很像，就多了useState，useEffect。`useState``useEffect`就是我们今天的主角Hook。

### useState

看上面注释1：

{% highlight javascript %}
const [num,setNum] = useState(0);    // 1
{% endhighlight %}

这种 JavaScript 语法叫数组解构。它意味着我们同时创建了 fruit 和 setFruit 两个变量，fruit 的值为 useState 返回的第一个值，setFruit 是返回的第二个值。

相比类组件的,`num`相当于`this.state.num`,`setNum`相当于`this.setState()`,当我们点击按钮的时候调用`setNum(num+1)`相当于调用`this.setState({num:this.state.num+1})`。

**useState为我们创建了当前函数组建的state,和改变state的方法并通过参数设置当时state的初始状态的值，这里设置`num=0`;

### useEffect

{% highlight javascript %}
    useEffect(()=>{         // 2
        console.log('组件已经加载成功+++++++');
    })
{% endhighlight %}

`useState`为函数组件提供了操作state的功能，`useEffect`为函数组件提供执行组件生命周期回调的功能。

在页面中同时执行组件App时，控制台打印如下

```code
组件已经加载成功+++++++  // 类组件打印
组件已经加载成功+++++++  // hook 函数组件
```
从上面打印可以看出来，`useEffect` 执行了函数componentDidMount生命周期。

我们分别点击不同组件的按钮，页面中组件数字都变成了1；但类组件并没有在控制台打印，而hook函数组件再次打印。
```code
组件已经加载成功+++++++  // hook 函数组件
```
从控制台表象上来看，每次hook类组件更新时都会触发useEffect函数；是不是很像类组件的componentDidUpdate。不要错，`useEffect`就是为函数组件提供componentDidMount和componentDidUpdate这个两个生命周期。但是从上面打印的情况来看的话，页面每次更新都执行``useEffect`有时并不是我们程序需要的我们怎么，例如想在componentDidMount中执行一次Ajax请求的话，我们会发现每次点击按钮都会发一次请求，在性能生会有很大的浪费，如果我们在ajax请求后在次调用`useState`改变页面数据，会发现页面进入了一个死循环。那么如何解决呢？

**uesEffect第二个参数**

`useEffect`还有第二个参数，它是 effect 所依赖的值数组。

{% highlight javascript %}
    useEffect(()=>{         // 2
        console.log('组件已经加载成功+++++++');
    },[num])
{% endhighlight %}

此时，只有当 num 改变后才会重新执行`useEffect`,那么对于上面只在componentDidMount中执行一次请求的问题，我们就可以通过传递一个参数控制，但是在Hook使用中，为了解决这个问题大部分同学都是直接传递一个[]空数组，虽然传递一个空数组是可以解决但是在某些情况下回出现一些无法更新的bug。

**useEffect处理副作用**

在我们编写代码的时候，通常会用到setInterval、订阅函数的方法等如下：
{% highlight javascript %}
    useEffect(()=>{         
        const timer = setInterval(()=>{ // do something },1000)
    },[])
{% endhighlight %}

一般情况为了防止内存泄漏我么都会在组件销毁时或者任务完成时将其销毁，hook函数组件通过return一个函数来清理这些副作用：

{% highlight javascript %}
    useEffect(()=>{         
        const timer = setInterval(()=>{ // do something },1000);
        return()=>{  // 在此执行清除副作用函数
            clearInterval(timer)   
        }
    },[])
{% endhighlight %}


### useContext
在开发过程中，我们一般通过props达到父子组件的信息传递，但是如果组件层级嵌套比较深，那传递props将会成为特别考验人们心智的东西。为此react社区也诞生了react-redux mobox等解决数据传递的问题。但是有时开发时我们并不需要维护复杂的action reducer，反而会使程序更加难以理解。我们仅仅只需要一个方法将公用的几个数据在不同组件之间传递。`context`就是这样的东西，`useContext`就是接收一个 context 对象并返回该 context 的当前值。用例如下：

{% highlight javascript %}
// context.js
    const Context = React.createContext();
    function ContextContainer(props){
        const [num,setNim] = setState(0);
        return(
            <Context.Provider value={[
                num,setNume
            ]}>
                { props.child }
            </Context.Provider>
        )
    }
    export { Context, ContextContainer }

// buttonAdd.js
    import {Context} from './context.js'
    export default function(){
        const [num,setNum] = useContext(Context);
        return(
            <div>
                <button onClick={()=>{setNum(num+1)}}>增加{num}</button>
            </div>
        )
    }

// buttonReduce.js
    import {Context} from './context.js'
    export default function(){
        const [num,setNum] = useContext(Context);
        return(
            <div>
                <button onClick={()=>{setNum(num-1)}}>减少{num}</button>
            </div>
        )
    }

// app.js
import {ContextContainer} from './context.js'
import Add from './buttonAdd.js'
import Reduce from './buttonReduce.js'
const App = () => {
  return (
    <div className="App">
      <ContextContainer>
        <Add />
        <Reduce />
      </ContextContainer>
    </div>
  );
};
{% endhighlight %}

当我们点击页面的增加按钮时候,减少按钮也发生增加，点击页面的减少按钮时候,减少按钮也发生减少。Add 和 Reduce之间共享num数据是如此简单。


### useReducer
它是`useState`的代替方案，在某些场景下，useReducer 会比 useState 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。
{% highlight javascript %}
const initialState = {count: 0};

function reducer(state, action) {
  switch (action.type) {
    case 'increment':
      return {count: state.count + 1};
    case 'decrement':
      return {count: state.count - 1};
    default:
      throw new Error();
  }
}

function Counter({initialState}) {
  const [state, dispatch] = useReducer(reducer, initialState);
  return (
    <>
      Count: {state.count}
      <button onClick={() => dispatch({type: 'increment'})}>+</button>
      <button onClick={() => dispatch({type: 'decrement'})}>-</button>
    </>
  );
}
{% endhighlight %}
### useRef
{% highlight javascript %}
const refContainer = useRef(initialValue);
{% endhighlight %}
useRef 返回一个可变的 ref 对象，其 .current 属性被初始化为传入的参数（initialValue）。返回的 ref 对象在组件的整个生命周期内保持不变。
{% highlight javascript %}
function TextInputWithFocusButton() {
  const inputEl = useRef(null);
  const onButtonClick = () => {
    // `current` 指向已挂载到 DOM 上的文本输入元素
    inputEl.current.focus();
  };
  return (
    <>
      <input ref={inputEl} type="text" />
      <button onClick={onButtonClick}>Focus the input</button>
    </>
  );
}
{% endhighlight %}
`useRef返回的对象在组件的生命周期是保持不变的。`
### useCallback
{% highlight javascript %}
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b],
);
{% endhighlight %}
把内联回调函数及依赖项数组作为参数传入 useCallback，它将返回该回调函数的 memoized 版本，该回调函数仅在某个依赖项改变时才会更新。当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。