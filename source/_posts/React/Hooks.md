---
title: React Hooks个人理解和总结
date: 2020-11-16 22:11:54
tags: React
---
# React Hooks是什么
  *取代类组件中的生命周期和状态，专门为函数式组件控制状态搭配的一套api*
# 类组件的缺陷
  * this指向问题(func.bind(this,’ccc’)  () => {this.func})
  * 在生命周期中需要添加或者删除副作用达到优化代码的效果(componentDidMount中注册和componentUnMount释放）
  * hoc和render props都需要嵌套一层类组件，造成层级冗余
  * Hooks可以解决上面的三个问题


# Hooks
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
  6.setState和useState的区别：setState会维护一个队列，如果多次setState会合并成一次进行渲染（这里是将新旧值进行合并然后在赋值），useState使用Object.is进行比较，如果值不变就不会重新渲染，如果值不同直接用新替换旧，并且useState不会自动合并更新对象,需要通过setObj({ ...obj, {value: obj.value + 1}})这样去合并
  ## useEffect

