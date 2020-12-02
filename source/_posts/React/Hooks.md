---
title: React Hooks个人理解和总结
date: 2020-11-16 22:11:54
tags: React
---
# React Hooks是什么
  *hooks开始是为了解决逻辑复用，为函数化专门提供的api*
# mixins
  使用分散加打平的方式将逻辑和状态混入组件中

* 命名空间冲突：属性名和方法名很容易和组件中冲突，导致被覆盖
* 很难判断多个mixins中属性从哪里来
* 需要了解整个mixins才能使用，维护困难

# HOC
  本质上就是一个纯函数，接收一个类，返回一个增强过后的类
  
* 整个类对于外部来说是不可控的，相当于是一个黑盒。
* ref操作dom被隔断。React.createRef
* 因为父组件无法访问子组件的state，导致无法用shouldComponentUpdate进行过滤渲染 React.pureComponent。
* 如果多层嵌套需要注意props上的属性覆盖问题
* 整体需要嵌套一层类组件


<pre>
<code class="language-js">
import React from 'react';
import withSecretToLife from 'components/withSecretToLife';
const DisplayTheSecret = props =>
```
(
  <div>
    The secret to life is {props.secretToLife}.
  </div>
);
```
const WrappedComponent = withSecretToLife(DisplayTheSecret);
export default WrappedComponent;
import React from 'react';
const withSecretToLife = (WrappedComponent) => {
  class HOC extends React.Component {
    render() {
      ```
      return (<WrappedComponent secretToLife={42} {...this.props}/>);
      ```
    }
  }
  return HOC;
};
export default withSecretToLife;
</code>
</pre>


# Render props
把dom和state通过render给你，你帮我用属性或者方法直接渲染出来
render props可以解决除了最后一条所有的问题

* 通过render属性自己控制state渲染
* 多层嵌套，在一个组件中命名就不会冲突
* 区别在于可不可以直接传递dom

<pre>
<code class="language-js">
import React from 'react';
const SharedComponent extends React.Component {
  state = {...}
  render() {
    ```
    return (<> {this.props.render(this.state)} </>);
    ```
  }
}
export default SharedComponent;
import React from 'react';
import SharedComponent from 'components/SharedComponent';
const SayHello = () => 
```
(<SharedComponent render={(state) => (
  <span>hello!,{...state}</span>
)}/>);
```
</code>
</pre>
# Hooks

## 优点

* 解决了hoc和render props需要嵌套一层的问题，(props传递加useRef)
* 解决了mixins中状态和方法来源不清和命名的问题(自定义hooks)
* 解决了类组件中将逻辑分散到各个生命周期，或者需要释放内存的操作，(useEffect和监听依赖)
* 类组件改函数组件，类组件有自己的作用域，会降低性能

## 缺点

* 必须在函数头部进行声明，无法在for循环或者if判断中进行声明
* 需要自己指定依赖，就算有eslint-plugin-react-hooks这种类似的插件，也会有警告，包括无效的依赖存在

## useState

  1.定义:
<pre>
<code class="language-js">
const [state, setState] = useState(0);
</code>
</pre>
  2.每次渲染都会具有独立的props和state，不同于类组件需要将state挂载到this上（每次渲染是独立的，属性也是独立的，函数需要重新创建），我们来看两个例子
<pre>
<code class="language-js">
// 父组件的count每隔一秒加1
handleClick = () => { // 类组件
    setTimeout(() => {
        alert(this.props.count); // 三秒结束之后输出最新值3
    }, 3000);
};
handleClick = () => { // hooks组件
    setTimeout(() => {
        alert(props.count); // 三秒结束之后输出初始值0
    }, 3000);
};
</code>
</pre>
由此，我们可以得出结论：hooks组件在每次渲染的时候都有自己的props。
3.我们将上面的props变成state试下
<pre>
<code class="language-js">
const Example = () => {
  const [count,setCount] = useState(0);
  handleClick = () => {
      setTimeout(() => {
          alert(count);
      }, 3000);
  };
  handleCountClick = () => { // 类组件
      setCount(count + 1);
  };
  ```
  return (
      <Fragment>
          <p>{count}</p>
          <button onClick={handleCountClick}>
              setCount
          </button>
          <button onClick={handleClick}>
              Delay setCount
          </button>
      </Fragment>);
  ```
}
</code>
</pre>
  我们先点击下面的Delay setCount，然后再点击上面的setCount，会发现p中的count变成了最新的值，而alert出来的还是初始值0
  4.所以在每次render的时候state也有自己的初始值
  5.setState和useState的区别：setState会维护一个队列，如果多次setState会合并成一次进行渲染（异步），在受控的情况下是异步的，setTimeout或者操作DOM的时候是同步的，useState使用Object.is进行比较，如果值不变就不会重新渲染，如果值不同直接用新替换旧，而且useState总是异步的
  ## useEffect
  1.定义：
<pre>
<code class="language-js">
useEffect(() => {}, [dep]);
</code>
</pre>
2.理解：在DOM渲染之后进行一些操作，相当于vue的mounted。
一般进行一些副作用操作，像发请求，对DOM操作，定时器等等，每一次effect之后都会重新render，所以把定时器放在effect里边，重新render之后自动释放，不会造成内存泄露
## useLayoutEffect
会在浏览器 layout 之后，painting 之前执行，useEffect会在painting之后执行
<pre>
<code class="language-js">
const LayoutDemo = () => {
  const [count, setCount] = React.useState(0);
  React.useEffect(() => {
    // 耗时 300 毫秒的计算
    const start = +new Date();
    while (300 >= (+new Date() - start)) {
      continue;
    }
    if (count === 0) {
      setCount(Math.random());
    }
  }, [count]);
  const handleClick = React.useCallback(() => setCount(0), []);
  ```
  return <button onClick={handleClick}>{count}</button>;
  ```
};
export default LayoutDemo;
</code>
</pre>

  ## useMemo && useCallback
  1.不同于useEffect在DOM渲染之后执行，useMemo在渲染中进行useMemo -> render -> useEffect，看下下边的例子：
<pre>
<code class="language-js">
const nameList = ['apple', 'peer', 'banana', 'lemon'];
var aaa = 1;
const MemoExample = (props: any) => {
  const [price, setPrice] = useState(0)
  const [name, setName] = useState('apple')
  let getProductName = () => {
      console.log('getProductName触发')
      return name
  };
  // 只对name响应
  useEffect(() => {
      console.log('name effect 触发')
  }, [name])
  // 只对price响应
  useEffect(() => {
      console.log('price effect 触发')
  }, [price])

  // memo化的getProductName函数
  const memo_getProductName = useMemo(() => {
      console.log('name memo 触发')
      return () => name  // 返回一个函数
  }, [name]);
  ```
  return (
      <Fragment>
          <p>{name}</p>
          <p>{price}</p>
          <p>普通的name：{getProductName()}</p>
          <p>memo化的：{memo_getProductName ()}</p>
          <button onClick={() => setPrice(price+1)}>价钱+1</button>
          <button onClick={() => setName(nameList[Math.random() * nameList.length << 0])}>修改名字</button>
      </Fragment>)
  ```
}
export default MemoExample
</code>
</pre>

注意:如果只想在name改变的时候重新渲染，需要使用memo做下渲染元素的优化
通过观察我们可以发现useMemo是在渲染中执行的，所以不要在内部执行渲染无关的操作(比如setState等等，会重复渲染页面，或者执行其他回调)
2.useMemo有以下三种情况用到：
  ① 进行某个值的计算，相当于vue的computed
<pre>
<code class="language-js">
const memoizedValue = useMemo(() => (a + b), [a, b]);
</code>
</pre>
  ② 用来缓存元素，相当于React.pureComponent，主要是防止不相关元素对复杂元素的重新渲染
  [使用useMemo缓存React Elements](https://codesandbox.io/s/goofy-grothendieck-tkmow?fontsize=14&hidenavigation=1&module=%2Fsrc%2FExample.js&theme=dark)
  ③ 缓存传入对象，避免子组件多次渲染(如果有函数传入的话使用useCallback)
     现在有多个按钮，每个按钮都共享一个状态count，我们用React.memo缓存每个button，期望点击每个按钮都只渲染一次，代码如下：
<pre>
<code class="language-js">
const CountButton = React.memo(function CountButton({onClick, count}) {
  ```
  return <button onClick={onClick}>{count}</button>
  ```
})

function DualCounter() {
  const [count1, setCount1] = React.useState(0)
  const increment1 = () => setCount1(c => c + 1)

  const [count2, setCount2] = React.useState(0)
  const increment2 = () => setCount2(c => c + 1)
  ```
  return (
    <>
      <CountButton count={count1} onClick={increment1} />
      <CountButton count={count2} onClick={increment2} />
    </>)
  ```
}
</code>
</pre>

我们预想的是在缓存了页面之后CountButton不会重新渲染，但是我们点击之后button的组件依旧会渲染两次，原因是React Hooks每次渲染都是相互独立的，传入的函数因为没有做缓存，每次传入的都是一个新的函数对象，所以才会渲染两次，我们使用useCallback包括函数const increment1 = useCallback() => setCount1(c => c + 1), [count1]);这里最好指定下依赖。
2.当我们需要在DOM频繁变化的时候去请求接口（比如文本框输入的同时自动搜索），需要npm安装 npm install use-async-memo --save
还可以加上防抖的逻辑：
<pre>
<code class="language-js">
  const [input, setInput] = useState()
  const [debouncedInput] = useDebounce(input, 300)
  const users = useAsyncMemo(async () => {
    if (debouncedInput === '') return []
    return await apiService.searchUsers(debouncedInput)
  }, [debouncedInput], [])
</code>
</pre>

## useReducer

1.定义:
const [state,dispatch] = useReducer(reducer,state)
2.调用:dispatch({type: 'add'})
3.使用场景：useState的替代方案，当一次动作需要多个state一起变化的时候使用，目的是将做什么(dispatch)和怎么做(action)分开，将表现（UI）和逻辑（业务分开），并且useReducer可以避免setState的多次渲染
4.官方示例:
<pre>
<code class="language-js">
  // 官方 useReducer Demo
  // 第一个参数：应用的初始化
  const initialState = {count: 0};

  // 第二个参数：state的reducer处理函数
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

  function Counter() {
      // 返回值：最新的state和dispatch函数
      const [state, dispatch] = useReducer(reducer, initialState);
      ```
      return (
          <>
              // useReducer会根据dispatch的action，返回最终的state，并触发rerender
              Count: {state.count}
              // dispatch 用来接收一个 action参数「reducer中的action」，用来触发reducer函数，更新最新的状态
              <button onClick={() => dispatch({type: 'increment'})}>+</button>
              <button onClick={() => dispatch({type: 'decrement'})}>-</button>
          </>);
      ```
  }
</code>
</pre>

## useContext

1.定义:
<pre>
<code class="language-js">
  const ThemeContext = React.createContext(value);
  ```
  <ThemeContext.Provider value={themes.dark}>
    <Toolbar />
  </ThemeContext.Provider>
  ```
</code>
</pre>
2.使用:创建一个context对象，初始化value，通过context的provider将value传入ThemeContext组件中，在ThemeContext中使用useContext获取，主要跨组件之间状态的共享
3.当传入的value变化，context会重新渲染，不管组件是否memo过
4.示例:
<pre>
<code class="language-js">
import React, { useContext } from "react";
import ReactDOM from "react-dom";

const TestContext= React.createContext({});

const Navbar = () => {
  const { username } = useContext(TestContext)

  ```
  return (
      <div className="navbar">
        <p>{username}</p>
      </div>
  )
  ```
}

const Messages = () => {
  const { username } = useContext(TestContext)

  ```
  return (
    <div className="messages">
      <p>1 message for {username}</p>
    </div>
  )
  ```
}

function App() {
  ```
  return (
    <TestContext.Provider
      value={{
        username: 'superawesome',
      }}
    >
      <div className="test">
        <Navbar />
        <Messages />
      </div>
    <TestContext.Provider/>
  );
  ```
}

const rootElement = document.getElementById("root");
```
ReactDOM.render(<App />, rootElement);
```
</code>
</pre>
5.结合useReducer和useContext，组件可以改变一些公共的state，一般是同步的需要渲染的
<pre>
<code class="language-js">
const initialState = 0;
function reducer(state=initialState,action){
    switch(action.type){
        case 'ADD':
            return {number:state.number+1};
        default:
            break;
    }
}
function SubCounter(){
    const {state, dispatch} = useContext(CounterContext);
    ```
    return (
        <>
            <p>{state.number}</p>
            <button onClick={()=>dispatch({type:'ADD'})}>+</button>
        </>
    )
    ```
}

function Counter(){
    const [state, dispatch] = useReducer((reducer), initialState, ()=>({number:initialState}));
    ```
    return (
      <>
        <CounterContext.Provider value={{state, dispatch}}>
            <SubCounter/>
        </CounterContext.Provider>
        </>
    )
    ```
}
</code>
</pre>

使用context共享reducer的方法和初始状态，让provider中所有的组件都可以使用
6.注意：每次context数据改变子组件都会重新渲染，相比redux和mobx会有性能问题

## 自定义hooks

取代hoc和render props，将公共逻辑写在自定义hooks中
比如我们现在有多个弹框，弹框中有select框，input框，整体是一个form表单，datepicker这种等等，我们可以把获取下拉框数据和提交表单封装成公共的逻辑放在自定义hooks中，也可以把每个弹框都定义成一个hook，用来配合UI初始化语法类似于这样:
<pre>
<code class="language-js">
const [state, setState, submitFn, ] = useModal();
</code>
</pre>

# 总结

* 与类组件不同的是，使用react hooks每次渲染都是独立的，并不会记住前一次渲染的结果，使用useRef.current去记住原先的状态
* 所有hooks都是根据依赖来执行回调，避免将逻辑分散在不同的生命周期中，所以依赖的选择至关重要
* 使用useEffect在DOM渲染之后进行一些副作用操作，比如请求数据等等
* useMemo和useCallback用来缓存对象函数或者DOM元素，减少元素的渲染次数
* useReducer和useContext配合在几个组件中共享state，如果是全局store还是建议用redux
* 自定义hooks用来封装公共的逻辑和组件行为


参考文章:
       [这一次彻底搞定useReducer-使用篇](https://www.jianshu.com/p/566f0d79ca7b)
       [React Hook之useContext的介绍与使用](https://blog.csdn.net/weixin_43606158/article/details/100750602)
       [React Hooks 详解 【近 1W 字】+ 项目实战](https://juejin.cn/post/6844903985338400782#heading-14)
       [React Hooks 最佳实践](https://juejin.cn/post/6844904165500518414#heading-11)
       [使用React hooks实现Vue的“计算属性”](https://zhuanlan.zhihu.com/p/80607611)
       [useMemo和useEffect有什么区别？怎么使用useMemo](https://www.jianshu.com/p/94ace269414d)
       [React组件Render Props VS HOC 设计模式](https://www.imooc.com/article/79154)
       [什么时候使用 USEMEMO 和 USECALLBACK](https://www.freesion.com/article/5948405874/)
