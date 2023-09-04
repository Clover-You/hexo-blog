title: React学习笔记
categories: React
tags: [前端,React,学习笔记]
date: 2022-01-02 17:57:06
---
# React入门

React由Facebook开发，它是一个用于构建用户界面/视图的JavaScript库

- 起初由Facebook的软件工程师Jordan Walke创建
- 于2011年部署于Facebook的newsfeed
- 随后在2012年部署于Instagram
- 2013年5月宣布开源。



### 为什么要学React？

- 原生JavaScript操作DOM繁琐、效率低 (**DOM-API操作UI**)

  ```javascript
  document.getElementById('...');
  ```

- 使用JavaScript直接操作DOM，浏览器会进行大量的**回流重绘**

- 原生JavaScript没有**组件化/模块化**的编码方案，代码复用率低。



### React的特点

- 采用**组件化/模块化模式**、**声明式编码**，提高开发效率及组件复用率。
- 在React Native中可以是用React与法进行**移动端开发**
- 使用**虚拟DOM**和优秀的**Diffing**算法，尽量减少与真实DOM的交互。



## 引包

```
react.development.js ===>> React核心库
react-dom.development.js ===>> React扩展库，用于支持react操作dom
prop-types.js ===>> 类型检查库
babel.min.js ===>> 语法转换核心库，jsx 转 js 
```

```html
<!-- 引入react核心库 -->
<script src="../js/react.development.js"></script>
<!-- 引入react扩展库，用于react支持操作dom -->
<script src="../js/react-dom.development.js"></script>
<!-- 引入babel，用于将es6转es5和jsx转js -->
<script src="../js/babel.min.js"></script>
```

注意编写代码的script标签一定要指定类型

- `type="text/babel"` 告诉浏览器使用babel来翻译

```html
<script type="text/babel"></script>
```



## Hello React

先准备好一个容器，React会将虚拟dom渲染到这个容器中

```html
<div id="app"></div>
```

创建一个虚拟dom，注意此处`<h1>...</h1>`不需要使用引号引起来。

```jsx
// 创建虚拟dom
const vdom = <h1>Hello React!</h1>;
```

将虚拟dom渲染到容器上

```jsx
// 渲染虚拟dom
ReactDOM.render(vdom, app);
```

启动走你～～

![HelloReact](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-08-17%2015.09.50.png)



## 创建虚拟DOM的第二种方式

在React中，除了使用jsx创建虚拟DOM外，还可以以使用js来进行创建。这需要使用到React核心库提供的`React.createElement`API进行创建。它有三个参数：

- 第一个参数：标签名
- 第二个参数：标签属性
- 第N个参数：标签内容

```html
<!-- 引入react核心库 -->
<script src="../js/react.development.js"></script>
<!-- 引入react扩展库，用于react支持操作dom -->
<script src="../js/react-dom.development.js"></script>
<div id="app"></div>

<script>
  const attr = {id: 'hello'}
  // 创建虚拟dom
  const vdom = React.createElement('h1',attr , 'Hello React!');
  // 渲染虚拟dom
  ReactDOM.render(vdom, app);
</script>
```



## 虚拟DOM与真实DOM

- 虚拟DOM它本质是一个Object类型的对象（一般对象）
- 虚拟DOM比较“轻”，而真实DOM比较“重”，因为虚拟DOM是React内部在用，用不上真实DOM上那么多的属性。
- 虚拟DOM最终会被React转化为真实DOM呈现在页面上。





## JSX

- 全称叫JavaScript XMl

- JSX是React定义的一种类似于XML的js拓展语法：JS + XML

- 本质是`React.createElement(component, props, ...children)`方法的语法糖。

- JSX用于简化创建虚拟DOM

  - 写法

    ```jsx
    const ele = <h1>Hello JSX!</h1>;
    ```

  - 他不是字符串，也不是HTML/XML标签

  - 它最终产生的就是一个JS对象

- 标签名可以是HTML标签，也可以是其他标签。



### JSX语法规则

1. 定义虚拟DOM的时候不要写引号

   ```jsx
   const vdom = (
       <h1 id="clover">
           <span>Hello React!</span>
       </h1>
   );
   ```

2. 标签中要混入JS表达式时需要使用`{}`

   ```jsx
   const author = "clover";
   const body = "hello world!";
   // 创建虚拟dom
   const vdom = (
     <h1 id={author}>
       <span>{body}</span>
     </h1>
   );
   ```

3. 样式的类名指定不要使用`class`，需要使用`className`

   ```jsx
   <style>
     .myStyle {
       background-color: bisque;
       font-size: 20px;
     }
   </style>
   const vdom = (
     <h1 className="myStyle" id="clover">
       <span>hello world!</span>
     </h1>
   );
   ```

4. jsx中，若想使用内联样式，不能使用html中的语法`style="color: #900"`，需要传递一个**json**到style属性中。

   ```jsx
   const vdom = (
     <h1 id="clover">
       <span style={{color: '#900', fontSize: '20px'}}>Hello React!</span>
     </h1>
   );
   ```

5. 虚拟DOM必须只有一个根标签，不允许出现多个根标签，例如：

   ```xml
   const vdom = (
     <h1 id="clover">..</h1>
     <h1 id="clover">..</h1>
   );
   ```

6. 必须有闭合标签或自闭和

7. 标签命名

   - 若是小写字母开头，则将该标签转为html中同名元素，若html中无该标签，则抛出异常。
   - 若大写字母开头，React就去渲染对应的组件，若找不到该组件，则抛出错误。



## 模块与组件



### 模块

- 对外提供特定功能的js程序，一般就是一个js文件
- 随着业务逻辑的增加，代码越来越多且复杂，为了提高代码质量、复用性、运行效率和阅读性，所以将不同功能点抽取成一个个模块。
- 当应用的js都是以模块的方式来编写时，那么这个应用就可以被定义为一个模块化的应用。



### 组件

- 组件是用来实现局部功能效果的代码和资源的集合
- 随着界面功能越来越复杂，组件化的方式能够提高代码质量、复用性和阅读性。
- 当应用是以多组件的方式实现，那么这个应用就会被定义为一个组件化的应用。



# React面向组件编程

## 函数式组件

定义函数式组件，函数名就是你的组件名，**注意首字母一定要大写，并且函数必须有返回值**

```jsx
function FnComponent() {
  return <div>我是函数式组件</div>;
}
```

将这个组件渲染到页面上

```jsx
ReactDOM.render(<FnComponent/>, app);
```

在这里时注意，`FnComponent`组件中的this是一个undefined，因为在经过babel翻译之后开启了严格模式。

在React执行了`ReactDOM.render(<FnComponent/>, app);`之后发生了两件事：

1. React解析组件标签，找到`FnComponent`组件
2. 发现组件时使用函数定义的，随后调用该函数将返回的虚拟DOM转为真实DOM呈现在页面中。



### props

如果是函数式组件，React会将props收集为参数传递到这个函数中

```jsx
function Test(props) {
  const { title } = props;
  return (
    <h1>{title}</h1>
  );
}
```

渲染组件

```jsx
ReactDOM.render(<Test title="Hello World"/>, app);
```



### props类型限制

给组件挂载一个`propTypes`属性，指定要限制的属性类型

```jsx
Test.propTypes = {
  title: PropTypes.string
}
```

若还需要指定该属性必传，可以使用`isRequired`进行限制

```jsx
Test.propTypes = {
  title: PropTypes.string.isRequired
}
```



### props默认值

当一个属性不是必传时，可以指定这个属性的默认值，这需要在组件上挂载`defaultProps`属性

```jsx
Test.defaultProps = {
  title: 'Hello World!'
}
```



## 类式组件

创建一个类，首字母要大写并且继承`React.Component`，在这个类中，必须有一个render方法，并且这个方法必须返回一个虚拟DOM。

```jsx
class ClassComponent extends React.Component{
  render() {
    return <div>你好</div>;
  }
}
```

渲染组件

```jsx
ReactDOM.render(<ClassComponent/>, app);
```

在React执行了`ReactDOM.render(<ClassComponent/>, app);`之后发生了两件事：

1. React解析组件标签，找到`FnComponent`组件
2. 发现组件时使用类定义的，随后new出来该类的实例。并通过该实例调用到原型上的render方法。
3. 将render方法返回的虚拟DOM转为真实DOM呈现在页面中。

在继承了`React.Component`之后，`ClassComponent`也将`React.Component`中的三大属性继承过来

- `props`
- `refs`
- `state`



### 组件三大核心组件



#### state

- state是组件对象最重要的属性，值是一个一般对象(Object)
- 组件被称为“状态机”，通过更新组件的state来更新对应的页面显示(重新渲染组件)

创建一个有状态组件

```jsx
class Weather extends React.Component {
  render() {
    return <h1>今天天气很炎热！</h1>
  }
}
ReactDOM.render(<Weather />, app);
```

初始化组件状态，一个组件可能会有多个状态，所以state都会定义为一个Object

```jsx
class Weather extends React.Component {
  constructor(props) {
    super(props);
    this.state = { isHot: true }
  }
}
```

让虚拟DOM里的`今天天气很炎热！`根据状态改变

```jsx
render() {
  return <h1>今天天气很{this.state.isHot ? '炎热' : "凉爽"}！</h1>;
}
```

添加一个点击事件，点击后修改`isHot`状态。在react中，状态不能直接修改，例如`this.state=xxx`，需要通过React内置的API`setState`进行修改，它修改的值会和原来的状态进行合并，形成一个新的状态对象。

```jsx
class Weather extends React.Component {
  constructor(props = {}) {
    super(props);
    this.state = { isHot: true };
    // 为changeWeather指定this对象
    this.changeWeather = this.changeWeather.bind(this);
  }

  render() {
    return (
      <h1 onClick={this.changeWeather}>
        今天天气很{this.state.isHot ? "炎热" : "凉爽"}！
      </h1>
    );
  }

  changeWeather() {
    // 通过setState修改状态
    this.setState({ isHot: !this.state.isHot });
    // this.state.isHot = !this.state.isHot;
  }
}
```



#### props

- 每个组件对象都会有props(properties)属性
- 组件标签的所有属性都保存在props中
- props是只读的，不允许修改，若尝试修改，React会抛出异常并阻断后续执行。



假如有这么一个需求：自定义一个用来显示一个人员信息的组件：

1. 姓名必须指定，且为字符串类型
2. 性别为字符串类型，如果性别没有指定，默认为“男”
3. 年龄必须指定，且为数字类型

先定义一个类式组件：

```jsx
class Person extends React.Component {
  render() {
    return (
      <ul>
        <li>姓名: </li>
        <li>性别: </li>
        <li>年龄: </li>
      </ul>
    );
  }
}
```

将props属性的值赋值到对应`li`标签中

```jsx
render() {
  const { name = "", age = '', sex = '' } = this.props;
  return (
    <ul>
      <li>姓名: {name}</li>
      <li>性别: {age}</li>
      <li>年龄: {sex}</li>
    </ul>
  );
}
```

props的值是外界调用这个组件时传递过来的

```jsx
ReactDOM.render(<Person name="clover" age="19" sex="男" />, app);
```

![Props](http://qiniu-note-image.ctong.top/note/images/%E6%88%AA%E5%B1%8F2021-08-18%2016.04.51.png)

##### propTypes

正常情况下，props可以接收随意数据类型，这种情况下很容易会导致某些错误，例如以上例子的年龄，如果通过传递过来的这个“19”来进行`+1`，最终结果是`191`，因为传递过来的是一个字符串的19而不是数字19。

若需要对props中属性的类型做限制，React 16以后需要引入`prop-types.js`依赖，并且，需要在该组件上挂载一个`propTypes`属性，这是强制性的。

```jsx
Person.propTypes = {}
```

对你需要做类型限制的属性进行指定，当传过来的属性类型不是你规定的类型时，在浏览器控制台会有错误，但这并不影响运行。

```jsx
Person.propTypes = {
  age: {
    type: PropTypes.number,
  },
}
```

如果需要指定这个属性必须存在，可以使用`PropTypes.isRequired`进行指定

```jsx
Person.propTypes = {
  age: PropTypes.number.isRequired,
};
```



> 如果类型是一个函数，那么这么写：`PropTypes.func`

##### defaultProps

`如果性别没有指定，默认为“男”`，默认值不能使用propTypes来规定，他只是用于规定类型。需要在组件中挂载`defaultProps`属性，在这个属性中定义属性默认值。

```jsx
Person.defaultProps = {
  sex: '男',
};
```



#### 字符串类形式refs

给组件内的标签定义ref属性来标识它，有点类似`id`属性，但他们又不同。

在原生js中，若想获取一个input的值，需要使用`document.getElementById('id').value`，在React中，可以使用`ref`直接获取到这个元素渲染后的**真实**DOM，看以下代码如何使用这个`refs`

创建一个组件

```jsx
class DemoComponent extends React.Component {
  buttonStyle = {
    margin: "20px",
  };

  showData() {...}

  render() {
    return (
      <div>
        <input ref="inp" type="text" name="userName" />
        <button style={this.buttonStyle} onClick={this.showData.bind(this)}>
          点我提示Input数据
        </button>
      </div>
    );
  }
}
```

当按钮点击的时候获取文本框的值并打印出来，`refs`保存的是一个`ref`集合，因为`ref`可以存在很多个。

```jsx
showData() {
  // refs 是一个对象
  const { inp } = this.refs;
  // 拿到name="userName"那个文本框的真实DOM
  console.log("DOM: ", inp);
  // 拿到文本框值
  console.log("value: ", inp.value);
}
```

以上属于字符串形式的ref，这种写法已经不被官方所推荐，后续更新可能随时会将其废弃。因为大量使用它会有效率问题。

> 如果你之前使用过 React，你可能了解过之前的 API 中的 string 类型的 ref 属性，例如 `"textInput"`。你可以通过 `this.refs.textInput` 来访问 DOM 节点。我们不建议使用它，因为 string 类型的 refs 存在 [一些问题](https://github.com/facebook/react/pull/8333#issuecomment-271648615)。它已过时并可能会在未来的版本被移除。



#### 回调形式refs

要定义回调形式的refs很简单，只需要将一个函数丢给`ref`属性即可，React会自动调用这个函数，并将当前节点的真实DOM通过回调的方式返回。这些操作在React将虚拟DOM转成真实DOM的时候触发。

可以将React返回的DOM保存在一个变量中

```jsx
<input
  ref={(currentNode) => (this.inpRef = currentNode)}
  type="text"
  name="userName"
  />
```

获取文本框的值只需要通过保存的这个ref即可

```jsx
this.inpRef.value
```



> 如果 `ref` 回调函数是以内联函数的方式定义的，在更新过程中它会被执行两次，第一次传入参数 `null`，然后第二次会传入参数 DOM 元素。这是因为在每次渲染时会创建一个新的函数实例，所以 React 清空旧的 ref 并且设置新的。通过将 ref 的回调函数定义成 class 的绑定函数的方式可以避免上述问题，但是大多数情况下它是无关紧要的。

**class 的绑定函数** 其实就是当前组件上挂载的方法，例如：

```jsx
saveInpRef = (currentNode) => {
  this.inpRef = currentNode;
};

render() {
  return (
    <div>
      <input ref={this.saveInpRef} type="text" name="userName" />
    </div>
  );
}
```

这里注意，不能使用`this.saveInpRef.bind(this)`，因为这已经是一个新的方法了  。



#### createRef API

这种方式是 React 新推出的，也是 React 官方推荐使用的一种方式。

`React.createRef()`  调用后可以返回一个容器，该容器可以存储被 `ref` 所标识的节点。

```jsx
class DemoComponent extends React.Component {

  inpRef = React.createRef()
  
  render() {
    return <input ref={this.inpRef} type="text" name="userName" />;
  }
}
```

可以通过 `this.inpRef.current` 来获取当前 DOM 。

```jsx
console.log(this.inpRef.current);
```





### 构造器与props

直接将官网摘过来吧~~~

```jsx
constructor(props)
```

**如果不初始化 state 或不进行方法绑定，则不需要为 React 组件实现构造函数。**

在 React 组件挂载之前，会调用它的构造函数。在为 React.Component 子类实现构造函数时，应在其他语句之前前调用 `super(props)`。否则，`this.props` 在构造函数中可能会出现未定义的 bug。

通常，在 React 中，构造函数仅用于以下两种情况：

- 通过给 `this.state` 赋值对象来初始化[内部 state](https://zh-hans.reactjs.org/docs/state-and-lifecycle.html)。
- 为[事件处理函数](https://zh-hans.reactjs.org/docs/handling-events.html)绑定实例

在 `constructor()` 函数中**不要调用 `setState()` 方法**。如果你的组件需要使用内部 state，请直接在构造函数中为 **`this.state` 赋值初始 state**：



```jsx
constructor(props) {
  super(props);
  // 不要在这里调用 this.setState()
  this.state = { counter: 0 };
  this.handleClick = this.handleClick.bind(this);
}
```

只能在构造函数中直接为 `this.state` 赋值。如需在其他方法中赋值，你应使用 `this.setState()` 替代。

要避免在构造函数中引入任何副作用或订阅。如遇到此场景，请将对应的操作放置在 `componentDidMount` 中。

> 注意
>
> **避免将 props 的值复制给 state！这是一个常见的错误：**
>
> ```jsx
> constructor(props) {
> super(props);
> // 不要这样做
> this.state = { color: props.color };
> }
> ```
>
> 
>
> 如此做毫无必要（你可以直接使用 `this.props.color`），同时还产生了 bug（更新 prop 中的 `color` 时，并不会影响 state）。
>
> **只有在你刻意忽略 prop 更新的情况下使用。**此时，应将 prop 重命名为 `initialColor` 或 `defaultColor`。必要时，你可以[修改它的 `key`](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html#recommendation-fully-uncontrolled-component-with-a-key)，以强制“重置”其内部 state。
>
> 请参阅关于[避免派生状态的博文](https://zh-hans.reactjs.org/blog/2018/06/07/you-probably-dont-need-derived-state.html)，以了解出现 state 依赖 props 的情况该如何处理。



## 事件处理

- 通过 onXX 属性指定事件处理函数 (注意大小写) 
- React 使用的是自定义 (合成) 事件，而不是使用的原生 DOM 事件。这样做的目的是为了更好的兼容性，因为每个不同的浏览器事件可能都不一样。
- React 中的事件是通过事件委托方式处理的 (委托给组件最外层的元素) ，使用事件委托是为了更高效的处理事件。
- 通过 `event.target` 可得到发生事件的 DOM 元素对象。不要过度使用 `ref`



## 收集表单数据

- 受控组件
- 非受控组件

需求：定义一个包含表单的组件，输入用户名密码后，点击登录提示输入信息。

### 非受控组件

定义组件

```jsx
class Login extends React.Component {
  render() {
    return (
      <form>
        用户名：
        <input type="text" name="userName" />
        密码：
        <input type="text" name="pass" />
        <button>登录</button>
      </form>
    );
  }
}
```

使用refs，用于获取这两个`input`

```jsx
// 用户名Input
userNameRef = React.createRef();

// 密码Input
userPassRef = React.createRef();

render() {
  return (
    <form>
      用户名：
      <input type="text" ref={this.userNameRef} name="userName" />
      密码：
      <input type="text" ref={this.userPassRef} name="pass" />
      <button>登录</button>
    </form>
  );
}
```

在`form`表单提交之后alert用户名和密码

```jsx
handleSubmit() {
  const usrPass = this.userPassRef.current;
  const userName = this.userNameRef.current;
  alert(`用户名：${userName.value}, 密码：${usrPass.value}`);
  return false;
}
```

```jsx
<form onSubmit={this.handleSubmit.bind(this)}>
```



### 受控组件

给 `input` 添加一个 `onChange` 事件去监听值改变，改变后将值赋值到某个变量中就行了。

```jsx
class Login extends React.Component {

  state = {
    userName: '',
    userPass: ''
  }

  handleSubmit() {
    alert(`用户名：${this.state.userName}, 密码：${this.state.userPass}`);
    return false;
  }

  // 设置userName
  setUserName(event){
    const val = event.target.value;
    this.setState({userName: val})
  }

  // 设置userPass
  setUserPass(event){
    const val = event.target.value;
    this.setState({userPass: val})
  }

  render() {
    return (
      <form
        action="javascript: void 0;"
        onSubmit={this.handleSubmit.bind(this)}
      >
        用户名：
        <input type="text" onChange={this.setUserName.bind(this)} name="userName" value={this.state.userName}/>
        密码：
        <input type="text" onChange={this.setUserPass.bind(this)} name="pass" value={this.state.userPass}/>
        <button>登录</button>
      </form>
    );
  }
}
```

> 受控组件：受到了 `state` 的控制。相当于手动实现 Vue 中的双向绑定。



## 组件的生命周期

- 组件对象从创建到死亡他会经历特定阶段
- React 组件对象包含一系列钩子函数（生命周期回调函数），在特定的时刻调用。
- 在定义组件时，在特定的生命周期回调函数中做特定的工作。
- 生命周期勾子函数和 reander 方法一样，只需要声明在那 React 就可以自己去调用它。



> - `componentWillMount`
> - `componentWillReceiveProps`
> - `componentWillUpdate`
>
> 这些生命周期方法经常被误解和滥用；此外，我们预计，在异步渲染中，它们潜在的误用问题可能更大。我们将在即将发布的版本中为这些生命周期添加 “UNSAFE_” 前缀。（这里的 “unsafe” 不是指安全性，而是表示使用这些生命周期的代码在 React 的未来版本中更有可能出现 bug，尤其是在启用异步渲染之后。）



### 生命周期流程图

这个图是比较老的 React 的生命周期

![react生命周期(旧)](http://qiniu-note-image.ctong.top/note/images/react%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F(%E6%97%A7).png)

这是 React@17 之后的生命周期流程图

![react生命周期(新)](http://qiniu-note-image.ctong.top/note/images/react%E7%94%9F%E5%91%BD%E5%91%A8%E6%9C%9F(%E6%96%B0).png)

### componentWillMount

`componentWillMount` 在组件将要挂载时被 React 调用。也就是render第一次执行前。



### componentWillReceiveProps

`componentWillReceiveProps` 在父组件更新时触发。它可以接收一个参数，那就是父组件传过来的 `props` 

```jsx
componentWillReceiveProps(props) {
  console.log("新的Props ========>>> ", props);
}
```





### shouldComponentUpdate

`shouldComponentUpdate` 在 `this.setState()` 调用或父时被 React 调用，它必须有一个 `boolean` 返回值，如果返回 `true` ，表示允许更新状态，否则不允许更新状态。它可以接一个参数，这个参数是当前被修改的 `state` 

```jsx
// 是否允许更改状态
shouldComponentUpdate(currentState) {
  return true;
}
```



### componentWillUpdate

在组件将要更新时 `componentWillUpdate` 被 React 调用。

它可以接收3个参数，一个是 props，一个是 state，还有一个是 `getSnapshotBeforeUpdate` 勾子的返回值 snapshot。这两个参数分别是组件**更新前**的 props 、 state 和快照snapshot。

```jsx
componentDidUpdate(oldProps, oldState, snapshot) {
  console.log('=======>>> componentDidUpdate');
}
```





### componentDidMount

`componentDidMount` 会在组件挂载在页面上或者说 `render` 被 React 调用完成时， React 调用 `componentDidMount` 勾子。

写一个例子：

1. 让指定文本做显示/隐藏的渐变动画
2. 从完全可见到彻底消失，耗时2s
3. 点击 ‘kill’ 按钮从界面中卸载组件

像这种需求可以在 `componentDidMount` 勾子中使用定时器。点击 “kill” 按钮后清楚这个定时器。

```jsx
class Life extends React.Component {
  state = { opacity: 1 };

  timer = null;

  kill() {
    if (this.timer) {
      clearInterval(this.timer);
      this.timer = null;
    }
    // 卸载组件
    ReactDOM.unmountComponentAtNode(app);
  }

  h1Style() {
    return { opacity: this.state.opacity };
  }

  // 生命周期勾子：组件挂载到页面上时被 React 调用
  componentDidMount() {
    this.timer = setInterval(() => {
      let { opacity } = this.state;
      opacity -= 0.1;
      if (opacity <= 0) {
        opacity = 1;
      }
      this.setState({ opacity });
    }, 200);
  }

  render() {
    return (
      <div>
        <h1 style={this.h1Style.call(this)}>学不会啊学不会！</h1>
        <button onClick={this.kill.bind(this)}>kill</button>
      </div>
    );
  }
}
```



### componentWillUnmount

`componentWillUnmount` 会在组件**将要**被卸载时被 React 调用。

像上面的例子，定时器也可以在组件卸载时被清除

```jsx
// 组件将要卸载时被 React 调用
componentWillUnmount() {
  if (this.timer) {
    clearInterval(this.timer);
    this.timer = null;
  }
}
```



### getSnapshotBeforeUpdate

`getSnapshotBeforeUpdate` 在 `render` 函数执行之后被 React 调用。

它可以接收两个参数，第一个参数是更新之前的 props ，第二个参数是更新之前的 state 。



使用栗子：

保持当前滚动条状态。

```jsx
const sleep = (time) =>
  new Promise((resolve) => setTimeout(resolve, time));

const newsStyle = {
  height: "30px",
  color: "#fff",
};

const newsBox = {
  height: "200px",
  width: "400px",
  backgroundColor: "#900",
  overflowY: "auto",
};

class Demo extends React.Component {
  newsBoxRef = React.createRef();

  state = { newsArr: [] };

  componentDidMount() {
    this.addNews();
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    return this.newsBoxRef.current.scrollHeight;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    const node = this.newsBoxRef.current;
    node.scrollTop += (node.scrollHeight - snapshot)
  }

  // 添加新闻
  addNews() {
    setInterval(() => {
      const { newsArr } = this.state;
      const news = `新闻 - ${newsArr.length + 1}`;
      this.setState({ newsArr: [news, ...newsArr] });
    }, 1000);
  }

  handleNews() {
    return this.state.newsArr.map((item, index) => {
      return (
        <div key={index} style={newsStyle}>
          {item}
        </div>
      );
    });
  }

  render() {
    return (
      <div id="news" style={newsBox} ref={this.newsBoxRef}>
        {this.handleNews.call(this)}
      </div>
    );
  }
}
const app = document.getElementById("app");
ReactDOM.render(<Demo />, app);
```



## Diffing算法

- 在 React 中遍历列表时，key 最好不要使用 `index` ，因为当前状态中的数据发生变化时， React 会根据 **新数据** 生成 **新的虚拟 DOM** ，随后 React 使用 diff 算法对 **新虚拟DOM** 与 **旧虚拟 DOM** 比较，比较规则如下
  - 旧虚拟 DOM 中找到了与新虚拟 DOM 相同的 key
    1. 若虚拟 DOM 中内容没变，直接使用之前的真实 DOM
    2. 若虚拟 DOM 中内容变了，则生成新的真实 DOM，随后替换掉页面中之前的真实 DOM
  - 虚拟 DOM 中未找到与新虚拟 DOM 相同的 key，根据数据创建新的真实 DOM，随后渲染到页面
- 用 index 作为 key 可能会引发的问题
  1. 若对数据进行逆序添加、逆序删除等破坏顺序操作，会产生没有必要的真实 DOM 更新，界面可能没问题，但这效率低。
  2. 如果结构中还包含输入类的 DOM，会产生错误 DOM 更新，界面显示有问题
  3. 注意！如果不存在对数据的逆序添加、逆序删除等破坏顺序操作，仅用于渲染列表用于展示，使用 index 作为 key 是没有问题的。
- 在开发中最好使用每条数据唯一标识作为 key，比如id、手机号、身份证号、学号等唯一值。如果确定只是简单的展示数据，那么用 index 也是可以的。



# React 脚手架

[一个简单的“待办”案例](https://github.com/YouChuantong/react_demo_todo)

- React 脚手架是一个用来快速创建一个基于 React 库的项目模板项目
  - 包含了所有需要的配置（语法检查、jsx编译、devServer...等）
  - 下载好了所有相关的依赖
  - 可以直接运行一个简单效果
- React 提供了一个用于创建 React 项目的脚手架：**create-react-app**
- 项目的整体技术架构为：React + Webpack + es6 + eslint
- 使用脚手架开发的项目特点：模块化、组件化、工程化。

## 创建项目并启动

- 全局安装 React 脚手架：`yarn global add create-react-app` 
- 使用脚手架命令创建项目：`create-react-app hello-app`
- 进入项目文件夹：`cd hello-app`
- 启动项目：`yarn start`



```
.
├── node_modules 「依赖包」
├── README.md
├── package.json  「npm配置文件」
├── public 「静态资源文件夹」
├── src	「源码文件夹」
│   ├── index.js	「入口文件」
└── yarn.lock
```



## 一个简单的Hello组件

在 index.js 中引入 React 核心组件

```js
// 引入 React 核心库
import React from "react";
// 引入 ReactDOM 核心库
import ReactDom from "react-dom";
```

再创建一个 Hello 组件

```jsx
import React from "react";

class Hello extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello React!</h1>
      </div>
    );
  }
}

export default Hello;
```

在 index.js 文件中将这个组件引入并将它挂载到容器

```jsx
import Hello from "./Hello";

const root = document.getElementById("root");

ReactDom.render(<Hello />, root);
```

搞定。



## 功能界面的组件化编码流程

- 拆分组件：拆分组件、抽取组件
- 实现静态组件：使用组件实现静态页面效果
- 实现动态组件
  - 动态显示初始化数据
    1. 数据类型
    2. 数据名称
    3. 保存在哪个组件
  - 交互（从绑定事件监听开始）

# React ajax

- React 本身只关注界面，并不包含发送 ajax 请求的代码
- 前端应用需要通过 ajax 请求与后台进行交互（json数据）
- React 应用中需要继承第三方 ajax 库或自己封装
- 常用的 ajax 库
  - jQuery 比较重，如果需要另外引入不建议使用
  - axios 轻量级 http 库，这也是市面上最常用的 http 库
    1. 封装 XmlHttpRequest 对象的 ajax
    2. 支持 Promise
    3. 可以用在浏览器端和 NodeJS 服务端



# React 路由

下载：`yarn add react-router-dom`

## SPA

- 单页 Web 应用（Single page web application 简称 SPA）
- 整个应用只有一个完整的页面
- 修改页面中的**链接**不会刷新页面，只会做页面的**局部刷新**
- 数据都需要通过 ajax 请求获取，并在前端异步展现



## 路由

- 一个路由就是一个映射关系（key: value）
- Key 为路径，value 可能是 function 或 component



### 路由分类

1. 后端路由
   - value 是 function，用来处理客户端提交的请求
   - 注册路由： `router.get(path, function(req, res))`
   - 工作流程：当 node 接收到一个请求时，根据请求路径找到匹配的路由，调用路由中的函数来处理请求，返回响应数据。
2. 前端路由
   - 浏览器端路由：value 是 component，用于展示页面内容
   - 注册路由：`<Route path="/test" component={Test} >`
   - 工作流程：当浏览器的 path 变为 /test 时，当前路由组件就会变为 Test 组件



## react-router

- react-router 是 React 的一个插件库
- 用于实现一个 SPA 应用

## react-router 相关API

### 内置组件

- `<BrowserRouter>` 路由器

- `<HashRouter>` 路由器

- `<Route>` 路由

  - `path` 路由地址
  - `component` 组件
  - `exact` 是否开启严格匹配，默认使用的是模糊匹配。严格匹配不要随便开启，有时候开启严格匹配会导致无法匹配多级路由。

- `<Redirect>` 重定向。一般写在所有路由注册的最下方，当所有路由都无法匹配时，跳转到 Redirect 指定的路由。

  ```jsx
  <Switch>
    <Route exact path="/home/{id}" component={Home} />
    <Route path="/about" component={About} />
    <Redirect to="/home" />
  </Switch>
  ```

  

- `<Link>` 切换路由链接，类似 html 中的 `<a>` 标签

  - `to` 要跳转的地址
  - `children` 标签体内容：`<a>标签体内容</a>`

- `<NavLink>` 用法与作用和 Link 一致

  - `activeClassName` 路径匹配时追加的 class 名称，默认追加 active

- `Switch` 可以提高路由匹配效率，在不使用 `Switch` 的时候，可以匹配多个相同路径的路由，而 `Switch` 只要匹配到一个就不匹配了。

  - `children` 匹配规则

    ```jsx
    <Switch>
      <Route path="/home" component={Home} />
      <Route path="/about" component={About} />
    </Switch>
    ```

    

> 路由需要包裹在路由器中

### 路由组件props

- `history` 对象
- `match` 对象
- `withRouter` 函数



## 向路由组件传递params参数

- 通过 `:key` 的方式在路由中声明 params 

  ```jsx
  <Route path="/home/message/detail/:id/:title" component={MessageDetail} />
  ```

- 通过 restful 风格的 url 触发这个路由

  ```jsx
  <Link to="/home/message/detail/01/消息1">消息1</Link>
  ```

- 在路由组件 props 中默认有三个参数，其中有一个是 `match` ，params 参数就保存在里面。

  ```jsx
  render() {
    const params = this.props.match.params;
    console.log(params)
    ...
  }
  ```

  ```json
  {id: "01", title: "消息1"}
  ```

  



## 向路由组件传递search参数

- 正常注册路由

  ```jsx
  <Route path="/home/message/detail" component={MessageDetail} />
  ```

- 路由链接（携带参数）

  ```jsx
  <Link to="/home/message/detail?id=01&title=消息1">消息1</Link>
  ```

- search 参数被放在了 props 的 location 中，是以字符串 `?id=01&title=消息1` 的形式存放，需要解析。

  ```jsx
  const { search } = this.props.location;
  // 使用 querystring 库解析 search 参数
  const parse = qs.parse(search.slice(1));
  ```

  ```json
  {id: "01", title: "消息1"}
  ```





## 向路由组件传递state参数

- 正常注册路由

  ```jsx
  <Route path="/home/message/detail" component={MessageDetail} />
  ```

- 路由链接，正常情况下 `Link` 的 `to` 是一个字符串，但是如果需要传递 state 属性，需要传递一个对象，这个对象包含 state。

  ```jsx
  <Link
    to={{
      pathname: "/home/message/detail/01/消息1",
        state: { name: "clover" },
    }}
    >
    消息1
  </Link>
  ```

- state 参数被放在了 props 的 location 中

  ```jsx
  render() {
    console.log(this.props.location.state);
    return (
      <ul></ul>
    );
  }
  ```

  ```json
  {name: "clover"}
  ```



## BrowserRouter与HashRouter区别

- 底层原理不一样
  - BrowserRouter 使用的是 H5 的 history API，不兼容 IE9 及以下版本。
  - HashRouter 使用的是 URL 的哈希值。
- URL 表现形式不一样
  - BrowserRouter 的路径中没有 `#` ，例如：`localhost:3000/demo`
  - HashRouter 的路径包含 `#` ，例如：`localhost:3000/#/demo`
- 刷新后对路由 state 参数的影响
  - BrowserRouter 没有任何影响，因为 state 保存在 history 对象中。
  - HashRouter 刷新后会导致路由 state 参数的丢失





# Redux

- Redux 是一个专门用于做**状态管理**的JS库
- 它可以用在 React、Angular、Vue 等项目中，但使用它的人基本都与 React 配合使用 
- 集中式管理 React 应用中多个组件**共享**的状态。



## 在什么情况下需要用 Redux

- 某个组件的状态，需要让其他组件可以随时拿到的时候（共享）
- 一个组件需要改变另外一个组件的状态（通信）



## Redux 原理图

![redux原理图](http://qiniu-note-image.ctong.top/note/images/redux%E5%8E%9F%E7%90%86%E5%9B%BE.png)



## 创建 Reducer

- reducer 本质是一个函数，它需要接收两个参数，分别是：之前的状态（preState），动作对象（action）

  ```js
  function countReducer(preState, action) {
    return {};
  }
  export default countReducer;
  ```

- reducer 有两个作用：初始化状态和加工状态

- reducer 被第一次调用时，是 store 自动触发的，传递的 preState 是 undefined。

- 在 reducer 中，可以根据 action 传递过来的类型分别处理 state。

  ```js
  function countReducer(preState, action) {
    const { type, data } = action;
    const method = strategy[type];
    if (method !== void 0) return method(preState, data);
    // 如果没找到可能处于初始化状态
    return { count: 0 };
  }
  
  const strategy = {
    increment(preState, data) {
      return { ...preState, ...data };
    }
  };
  ```

  



## 创建Store

- 在 redux 目录下创建一个 store.js 文件

- 在这个文件中引入 redux 中的 createStore 函数，使用这个函数创建一个 reducer。

  ```js
  // 引入 redux
  // createStore 转门用于创建 redux 中最为核心的 store 对象。
  import { createStore } from "redux";
  ```

- createStore 调用时需要传入一个为其服务的 reducer

  ```js
  import coutReducer from "./count_reducer";
  const store = createStore(coutReducer);
  export default store;
  ```

  

## Redux 3个核心API

使用这些API需要引入创建好的 store

### getState

获取 redux 中被管理的状态（获取数据）

```jsx
const state = store.getState();
```



### dispatch

向 Redux 发布消息，需要携带 action，Store 会将这个 action丢给 Reducers 去处理。

```jsx
const { count } = store.getState();
store.dispatch({ type: "increment", data: { count: count + 1 } });
```



### subscribe

可以监测 Redux 中状态的改变。Redux 只负责管理状态，至于状态的改变驱动着页面的展示，需要我们自己实现。

```jsx
store.subscribe(() => {
  // 状态被改变啦
  this.setState({});
});
```



## 异步action

在store中，不止可以传一个一般对象，其实还可以传一个函数，在这个函数里面搞个异步任务就成。

默认情况下， store 不允许传除 Object 以外的参数，需要使用一个中间件 redux-thunk让 store 接受函数参数。

下载这个中间件

```
yarn add redux-thunk
```

在 store 中引入这个中间件

```js
import thunk from 'redux-thunk';
```

使用 Redux 提供的 applyMiddleware 去使用这个中间件，并且需要将其作为 createStore 的第二个参数。

```js
const thunkMiddleware = applyMiddleware(thunk);
const store = createStore(countReducer, thunkMiddleware);
```

在 actionCreator 中执行异步操作

```js
// 睡眠
const sleep = time =>
  new Promise((resolve, reject) =>
    setTimeout(resolve, time)
  );

export const createIncrementAsyncAction = state =>
  async (dispatch) => {
    // 使用 sleep 阻塞执行
    await sleep(1000);
    dispatch(createIncrementAction(state));
  }

const action = (type, data) => ({type, data});

```

 

# React-Redux

- 所有的 UI 组件都应该包裹一个容器组件，他们是父子关系
- 容器组件是真正和 redux 打交道的，里面可以随意使用 redux 的 api
- UI 组件中不能使用任何 redux 的 api
- 容器组件会传给 UI 组件
  1. redux 中所保存的状态
  2. 用于操作状态的方法



下载 `react-redux` 

```
yarn add react-redux
```



## 建立容器组件与UI组件的连接

先创建一个容易组件，用于连接 UI 组件。

引入 UI 组件

```js
import CountUI from "../../components/Count";
```

引入 react-redux 中的 connect 函数，用于连接 UI 组件。

```js
import {connect} from "react-redux"
```

连接容器组件与UI组件，connect 第一次被调用时可以传递两个参数

- 第一个参数是一个 `function` 类型，用于给 UI 组件传递状态，返回的对象中的 key 就作为传递给 UI 组件 props 的 key，value 就作为传递给 UI 组件 props 的 value。这个方法可以接收两个参数。这个方法还可以接收两个参数
  - 第一个参数是 redux 中的状态
  - 第二个参数是父组件给容器组件传递的 props
- 第二个参数是一个 `function` 类型，用于声明给 ui 组件操作状态的方法。返回的对象中的 key 就作为传递给 UI 组件 props 的 key，value 就作为传递给 UI 组件 props 的 value。这个方法同样可以接收两个参数
  - 第一个参数是 store 中的 `dispatch` 函数
  - 第二个参数是父组件传递给容器组件的 props

```js
/**
 * 返回一个Props
 * @param state redux 中的状态
 * @param props 属性
 * @returns {*} Props
 */
const createProps = function (state, props) {
  return state;
}

/**
 * 操作状态的方法
 * @param dispatch store 中的 dispatch 函数
 * @param props 父组件传递给容器的props
 * @returns {{}} ui props
 */
const reducers = function (dispatch, props) {
  return {
    increment(state) {
      dispatch(createIncrementAction(state));
    }
  };
}

export default connect(createProps, reducers)(CountUI);
```

使用这个组件时直接引入这个容器组件，还需要通过 props 给容器组件传递 store

```jsx
import { Component } from "react";
import Count from "./containers/Count";
import store from "./redux/store";

export default class App extends Component {
  render() {
    return <Count store={store} />;
  }
}
```

> 使用  react-redux 之后不需要再手动监测状态改变。~~store.subscribe();~~





## Provider组件

由于每个容器都需要提供一个 store ，如果有10个容器，那么就得写10次 `store={store}` 这种代码，而 Provider 组件可以帮我们做这件事，只需要给 Provider 组件提供一次 store，它就能精准的识别每一个容器，并将这个 store 传进去。

```jsx
ReactDOM.render(
  <Provider store={store}>
    <App/>
  </Provider>,
  container);
```