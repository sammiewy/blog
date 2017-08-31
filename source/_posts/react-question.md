---
title: reactJS中问题整理
date: 2016-06-21 20:13:59
tags: ReactJS
categories: React
layout: timeline
---

##### Q: reactjs为什么要设计成单向数据流？
**A**："在React中，数据流向是单向的——从父节点传递到子节点，因而组件是简单且易于把握的，他们只需从父节点获取props渲染即可。如果顶层组件的某个prop改变了，React会递归地向下遍历整棵组建树，重新渲染所有使用这个属性的组件。"

个人觉得这个解释不是很全面，搜到一个相关文章[React 单向数据流分析](http://karynsong.github.io/2016-02/38/)，但是讲得也不是很清楚， 我觉得可以看完Redux再回过来总结这个问题。


##### Q. 外部传入的props与getDefaultProps的关系，是谁覆盖谁？
**A:** 外部传入的props与getDefaultProps的关系是外部传入的props属性覆盖getDefaultProps。

```
var Hello = React.createClass({
  render: function() {
    return <ChildNode name="parentNode" />;
  }
});
var ChildNode = React.createClass({
 getDefaultProps: function() {
  	return {name: 'childNode'};
  },
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }
});

ReactDOM.render(
  <Hello />,
  document.getElementById('container')
);


//输出
Hello parentNode
```



##### Q. 生命周期的componentDidMount时期是否已经插入了DOM？
**A:** componentDidMount时期已经插入了DOM，

```
var Hello = React.createClass({
  render: function() {
    return <ChildNode name="parentNode" />;
  }
});
var ChildNode = React.createClass({
	getDefaultProps: function() {
  	console.log(123);
  	return {name: 'childNode'};
  },
  componentDidMount: function() {
  	var childNode = document.getElementById('childId');
    console.log(childNode);
  },
  render: function() {
    return <div id="childId">Hello {this.props.name}</div>;
  }
});

ReactDOM.render(
  <Hello />,
  document.getElementById('container')
);
```

控制台输出:

```
<div data-reactroot="" id="childId">
<!-- react-text: 2 -->
Hello 
<!-- /react-text -->
<!-- react-text: 3 -->
parentNode
<!-- /react-text -->
</div>
```

##### 4. componentWillReceiveProps的作用， 会继续执行render吗？
**A:**   在组件接收到新的 props 的时候调用。在初始化渲染的时候，该方法不会调用。
用此函数可以作为 react 在 prop 传入之后， render() 渲染之前更新 state 的机会。老的 props 可以通过 this.props 获取到。在该函数中调用 this.setState() 将不会引起第二次渲染。

```
var Hello = React.createClass({
    getInitialState: function () {
        return {name: 'parentNode'};
    },
    clickBtn: function () {
        this.setState({name: 'change ParentNode!'});
    },
    render: function () {
        return <div className={this.state.name}>
            <ChildNode name="parentNode"/>;
            <button onClick={this.clickBtn}>点击</button>
        </div>
    }
});
var ChildNode = React.createClass({
    getDefaultProps: function () {
        return {name: 'childNode'};
    },
    getInitialState: function () {
        return {message: 'welcome!'};
    },
    componentDidMount: function () {
        var childNode = document.getElementById('childId');
    },
    componentWillReceiveProps: function (nextProps) {
        console.log('componentWillReceiveProps---'+nextProps.name);
        console.log('componentWillReceiveProps---'+ this.props.name);
        console.log('componentWillReceiveProps---'+this.state.message);
        this.setState({message: 'change Welcome'});

    },
    render: function () {
        console.log('render---' + this.state.message);
        return <div id="childId">Hello {this.props.name} {this.state.message}</div>;
    }
});

ReactDOM.render(
    <Hello />,
    document.getElementById('container')
);

```
初始运行时：

```
页面输出：
Hello parentNode welcome!

制台输出:
render---welcome!
```
点击btn的是时：

```
页面值：
Hello parentNode change Welcome

控制台输出:
componentWillReceiveProps---parentNode
componentWillReceiveProps---parentNode
componentWillReceiveProps---welcome!
render---change Welcome
```

**官网总结：**

当节点初次被放入的时候 componentWillReceiveProps 并不会被触发。这是故意这么设计的？

原因是因为 componentWillReceiveProps 经常会处理一些和 old props 比较的逻辑，而且会在变化之前执行；不在组件即将渲染的时候触发，这也是这个方法设计的初衷。

##### 5. immutableJS是如何解决shouldComponentUpdate中复杂对象的比较的？
**A:**  
在shouldComponentUpdate分别判断props和state有没有变化。

shouldComponentUpdate中使用immutableJS代码：
```
import { is } from 'immutable';

shouldComponentUpdate: (nextProps = {}, nextState = {}) => {
  const thisProps = this.props || {}, thisState = this.state || {};

  if (Object.keys(thisProps).length !== Object.keys(nextProps).length ||
      Object.keys(thisState).length !== Object.keys(nextState).length) {
    return true;
  }

  for (const key in nextProps) {
    if (thisProps[key] !== nextProps[key] || is(thisProps[key], nextProps[key])) {
      return true;
    }
  }

  for (const key in nextState) {
    if (thisState[key] !== nextState[key] || is(thisState[key], nextState[key])) {
      return true;
    }
  }
  return false;
}
```
is方法是先比较两个对象是不是同一个地址， 再比较两个对象的valueOf，即两个对象的值上否相同。
immutableJS中is实现代码：
```
export function is(valueA, valueB) {
  if (valueA === valueB || (valueA !== valueA && valueB !== valueB)) {
    return true;
  }
  if (!valueA || !valueB) {
    return false;
  }
  if (typeof valueA.valueOf === 'function' &&
      typeof valueB.valueOf === 'function') {
    valueA = valueA.valueOf();
    valueB = valueB.valueOf();
    if (valueA === valueB || (valueA !== valueA && valueB !== valueB)) {
      return true;
    }
    if (!valueA || !valueB) {
      return false;
    }
  }
  if (typeof valueA.equals === 'function' &&
      typeof valueB.equals === 'function' &&
      valueA.equals(valueB)) {
    return true;
  }
  return false;
}
```


