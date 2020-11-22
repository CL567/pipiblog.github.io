---
title: React Hooks个人理解和总结
date: 2020-11-16 22:11:54
tags: React
---
# React Hooks是什么
  *hooks开始是为了解决逻辑复用，为函数化专门提供的api*
# mixins
  --使用分散加打平的方式将逻辑和状态混入组件中
  *命名空间冲突：属性名和方法名很容易和组件中冲突，导致被覆盖
  *很难判断多个mixins中属性从哪里来
  *需要了解整个mixins才能使用，维护困难
# HOC(我先给你个组件(dom)，你通过props增强之后返给我)
  *整个类对于外部来说是不可控的，相当于是一个黑盒。
  *ref操作dom被隔断。React.createRef
  *因为父组件无法访问子组件的state，导致无法用shouldComponentUpdate进行过滤渲染 React.pureComponent。
  *如果多层嵌套需要注意props上的属性覆盖问题
  *整体需要嵌套一层类组件
  网上摘抄的demo
  ```
  import React from 'react';
  const withSecretToLife = (WrappedComponent) => {
    class HOC extends React.Component {
      render() {
        return (
          <WrappedComponent
            secretToLife={42}
            {...this.props}
          />
        );
      }
    }
    return HOC;
  };
  export default withSecretToLife;


  import React from 'react';
  import withSecretToLife from 'components/withSecretToLife';
  const DisplayTheSecret = props => (
    <div>
      The secret to life is {props.secretToLife}.
    </div>
  );
  const WrappedComponent = withSecretToLife(DisplayTheSecret);
  export default WrappedComponent;
  ```
# Render props(我把dom和state通过render给你，你帮我用你的属性或者方法直接渲染出来)
  render props可以解决除了最后一条所有的问题
  *通过render属性自己控制state渲染
  *多层嵌套，在一个组件中命名就不会冲突
  *区别在于可不可以直接传递dom*
```
  import React from 'react';
  const SharedComponent extends React.Component {
    state = {...}
    render() {
      return (
        <div>
          {this.props.render(this.state)}
        </div>
      );
    }
  }
  export default SharedComponent;

  import React from 'react';
  import SharedComponent from 'components/SharedComponent';
  const SayHello = () => (
    <SharedComponent render={(state) => (
      <span>hello!,{...state}</span>
    )} />
  );
```
# Hooks
  ## 优点
  *解决了hoc和render props需要嵌套一层的问题，(props传递加useRef)
  *解决了mixins中状态和方法来源不清和命名的问题(自定义hooks)
  *解决了类组件中将逻辑分散到各个生命周期，或者需要释放内存的操作，(useEffect和监听依赖)
  *类组件改函数组件，类组件有自己的作用域，会降低性能
  ## 缺点
  *必须在函数头部进行声明，无法在for循环或者if判断中进行声明
  *需要自己指定依赖，就算有eslint-plugin-react-hooks这种类似的插件，也会有警告，包括无效的依赖存在
  ## useState
  1.定义:
  ```
  const [state, setState] = useState(0);
  ```
  2.每次渲染都会具有独立的props和state，不同于类组件需要将state挂载到this上（每次渲染是独立的，属性也是独立的，函数需要重新创建），我们来看两个例子
  ```
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
  ```
  由此，我们可以得出结论：hooks组件在每次渲染的时候都有自己的props。
  3.我们将上面的props变成state试下
  ```
  const Example = () => {
    const [count,setCount] = useState(0);
    handleClick = () => { // 类组件
        setTimeout(() => {
            alert(count); // 三秒结束之后输出三秒之前的count
        }, 3000);
    };
    handleCountClick = () => { // 类组件
        setCount(count + 1);
    };
    return (
        <div>
            <p>{count}</p>
            <button onClick={handleCountClick}>
                setCount
            </button>
            <button onClick={handleClick}>
                Delay setCount
            </button>
        </div>
    );
  }
  ```
  我们先点击下面的Delay setCount，然后再点击上面的setCount，会发现p中的count变成了最新的值，而alert出来的还是初始值0
  4.所以在每次render的时候state也有自己的初始值
  5.setState和useState的区别：setState会维护一个队列，如果多次setState会合并成一次进行渲染（这里是将新旧值进行合并然后在赋值），useState使用Object.is进行比较，如果值不变就不会重新渲染，如果值不同直接用新替换旧，并且useState不会自动合并更新对象,需要通过setObj({ ...obj, {value: obj.value + 1}})这样去合并
  ## useEffect
  1。定义：
  ```
  useEffect(() => {}, [dep]);
  ```
  2.理解：在DOM渲染之后进行一些操作，相当于vue的mounted。
  一般进行一些副作用操作，像发请求，对DOM操作，定时器等等，每一次effect之后都会重新render，所以把定时器放在effect里边，重新render之后自动释放，不会造成内存泄露
# useMemo
  1.用来缓存元素，比如大量的dom计算，如果state变化就会重新渲染，看下下边的例子：
  ```
  const nameList = ['apple', 'peer', 'banana', 'lemon']
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
  
    // memo化的getProductName函数   🧬🧬🧬
    const memo_getProductName = useMemo(() => {
        console.log('name memo 触发')
        return () => name  // 返回一个函数
    }, [name]);
    return (
        <Fragment>
            <p>{name}</p>
            <p>{price}</p>
            <p>普通的name：{getProductName()}</p>
            <p>memo化的：{memo_getProductName ()}</p>
            <button onClick={() => setPrice(price+1)}>价钱+1</button>
            <button onClick={() => setName(nameList[Math.random() * nameList.length << 0])}>修改名字</button>
        </Fragment>
    )
}
export default MemoExample
```
注意:如果只想在name改变的时候重新渲染，需要使用memo做下渲染元素的优化
想象以下的场景，现在有多个按钮，每个按钮都共享一个状态count，我们用React.memo缓存每个button，期望点击每个按钮都只渲染一次，代码如下：
```
const CountButton = React.memo(function CountButton({onClick, count}) {
  return <button onClick={onClick}>{count}</button>
})

function DualCounter() {
  const [count1, setCount1] = React.useState(0)
  const increment1 = () => setCount1(c => c + 1)

  const [count2, setCount2] = React.useState(0)
  const increment2 = () => setCount2(c => c + 1)

  return (
    <>
      <CountButton count={count1} onClick={increment1} />
      <CountButton count={count2} onClick={increment2} />
    </>
  )
}
```
我们预想的是在缓存了页面之后CountButton不会重新渲染，但是我们点击之后button的组件依旧会渲染两次，原因是React Hooks每次渲染都是相互独立的，传入的函数因为没有做缓存，每次传入的都是一个新的函数对象，所以才会渲染两次，我们使用useCallback包括函数const increment1 = useCallback() => setCount1(c => c + 1), [count1]);最好指定下依赖。
2.当我们需要fetchData变化来初始化一个函数时候，需要用useCallback
3.用来缓存数据，像上面的函数如果变成对象可以用useMemo缓存，vue中的计算属性也可以用useMemo来缓存，还有像useAsyncMemo这种，不过他需要npm安装 npm install use-async-memo --save
还可以加上防抖的逻辑：
```
  const [input, setInput] = useState()
  const [debouncedInput] = useDebounce(input, 300)
  const users = useAsyncMemo(async () => {
    if (debouncedInput === '') return []
    return await apiService.searchUsers(debouncedInput)
  }, [debouncedInput], [])
```
## useReducer
  