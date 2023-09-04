title: React react-redux props或state更新视图无法重新渲染问题
categories: 随笔
tags: [JavaScript,TypeScript,React]
date: 2022-01-02 15:34:21
---
记录学习React时自己是如何挖坑把自己埋了的过程：`children`以及其它`props`被修改时相关组件无法重新渲染（做了两天）

父组件代码：
```tsx
class UserHome extends Component<Props, State> implements IUserHome {

  public name: string | undefined;

  public readonly state: State = initialState;

  public handlerClick() {
    store.dispatch(sendAction());
  }

  /**
   * DOM挂载完成后执行
   */
  public componentDidMount() {
    store.subscribe(this.subscribe)
  }

  public render() {
    return (
      <HomeBackground url={BackgroundImg}>
        <HomeScreenHeightBox width={1400} background='rgba(246, 248, 249, .92)'>
          <button onClick={() => {
            this.props.setCount(10)
            // console.log(this.state.count)
          }}> 点我</button>
          <HomeNavigationBar height={this.state.count} />
        </HomeScreenHeightBox>
      </HomeBackground> 
    )
  }
}

const mapDispatchToProps = (dispatch: Function): mapDispatchToPropsInterface => {
  return {
    sendAction() {
      dispatch({
        type: 'send_action',
        value: "UpYou of blog"
      })
    },
    setCount(sun) {
      dispatch({
        type: 'set_count',
        count: sun
      })
    }
  }
}

const mapStateToProps = (state: StateInferface) => {
  // console.log(state)
  return state;
}

export default connect(mapStateToProps, mapDispatchToProps)(UserHome)
```

readux层
```tsx
const initialState: StateInferface = {
  value: "HELLO WORLD",
  count: 50,
  height: 50
}

const reducer = (state = initialState, action: ActionInterface): any => {
  switch (action.type) {
    case "send_action":
      return Object.assign({ ...state }, action);
    case "set_count": 
      return Object.assign({ ...state }, {  count: state.count + 10});
    default:
      return state;
  }
}

export default reducer;
```
现需求是点击“点我”按钮改变`HomeNavigationBar`组件`height`属性,`HomeNavigationBar`代码(这个组件实际上是将`height`再传给另一个容器组件`NavigationBar`,一下省略了中间调用代码)：
```tsx
/**
 * 首页导航栏容器
 */
export default class HomeNavigationBar extends PureComponent<Props, object> implements IHomeNavigationBar {

  // 设置默认Props
  static defaultProps = {
    height: 50,
    backColor: "#3a3f51"
  }

  private height?: number = this.props.height; // 导航栏高度
  private backColor?: string = this.props.backColor; // 背景颜色
  private children?: ReactNode = this.props.children; // 插槽

  render() {
    const homeNavigationBarStyles = {
      height: `${this.height}px`,
      backgroundColor: `${this.backColor}`
    }
    return (
      <div id='HomeNavigationBar' style={homeNavigationBarStyles}>
        {this.props.height}
        <div className="flex-1">
          {this.children}
        </div>
      </div>
    );
  }
}

```

在`NavigationBar`中有几段代码导致无法动态改变、重新渲染组件
![image.png](http://qiniu-note-image.ctong.top//note/images/202201021532823.png)
为了方便直接将`props`中的值给到`height`字段，来简单验证一下图中圈起部分代码的可行性：

![image.png](http://qiniu-note-image.ctong.top//note/images/202201021533565.png)

上图证明无法对基础数据类型的数据进行修改，而是直接将当前变量中的内存地址替换：
```js
let a = 0x213; // 例如等于3
a = 2;// 0x645 改变a的值，实际上改变的是内存地址
```
所以当`props`改变时`height`数据没变化就是这个原因,需要将`style`中的`this.height`改为`this.props.height`，声明的其它变量作废...
由于react中state或props改变时就会触发`render`方法重新渲染DOM，可以在`render`中定义`height`等字段`const { children, height, backColor } = this.props;`
```tsx
render() {
    const { children, height, backColor } = this.props;
    const homeNavigationBarStyles = {
      height: `${height}px`,
      backgroundColor: `${backColor}`
    }
    return (
      <div id='HomeNavigationBar' style={homeNavigationBarStyles}>
        <div className="flex-1">
          {children}
        </div>
      </div>
    );
  }
```