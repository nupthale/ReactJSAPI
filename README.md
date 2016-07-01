# 收获总结

&emsp;&emsp;说到react，都会想到的是virtualDOM, 高效的性能，不可变数据Immutable，数据流架构flux等；通过本次学习主要了解virtualDom到底是什么、react如何保证渲染的高效性以及为什么要使用Immutable数据;
针对上面提出的3个问题， 这里收集了3篇讲解非常清楚的文章，分别给予回答：

1. 到底什么是virtualDOM， 参考文章：[深度剖析：如何实现一个 Virtual DOM 算法 ](https://github.com/livoras/blog/issues/13)

2. react如果保证高效的性能，参考文章： [react如何性能达到最大化(前传)，暨react为啥非得使用immutable.js](https://segmentfault.com/a/1190000004290333)

3. 为什么要用ImmutableJs， 参考文章：[Immutable 详解及 React 中实践](https://github.com/camsong/blog/issues/3)

读过上面两篇之后， 下面以一个简单的例子，总结一下：
```
class Person extends React.Component {

  constructor(props, context) { ... }
  handleClick(crt) { ... }
  
  render() {
    var list = this.state.list;
    return (
        <div>
        {
          list.map((item) => <button onClick={()=>this.handleClick(item)}>test{item.active}</button>)
        }
        </div>
     )
  }
} 

ReactDOM.render(<Person/>, document.body);
```
render函数部分编译后如下：
```
Person.prototype.render = function render() {
    var _this2 = this;

    var list = this.state.list;
    return React.createElement(
      "div",
      null,
      list.map(function (item) {
        return React.createElement(
          "button",
          { onClick: function onClick() {
              return _this2.handleClick(item);
            } },
          "test",
          item.active
        );
      })
    );
  };
```
1. React.createElement定义如下：
    
    a. 返回一个ReactElement，VirtualDOM的一个节点；
    
    b. 第三个参数children表示该ReactElement下的孩子节点；
    ```
    ReactElement createElement(
      string/ReactClass type,
      [object props],
      [children ...]
    )
    ```

2. 从顶层的`ReactDOM.render(<Person/>, document.body);`开始render，就形成了一颗虚拟的DOM树， 如下图：
![image](http://haitao.nos.netease.com/47aad28d8cbf461baacf1ddd9f47d00f.png)
这样就构成了一个虚拟的DOM树， 也即上面提到的VirtualDom

3. react通过两部分保证组件的性能：
    
    a. 当调用this.setState方法时，react默认会触发组件的render调用，每次render方法会根据输入的新数据生成一个新的节点实例，react通过DIff算法比较二者，找到最小更新单元后， 把差异应用到真正的DOM树上；

    b. 当每次调用this.setState修改数据后，会依次调用以下生命周期方法：
    ```
    1、shouldComponentUpdate
    2、conponentWillUpdate
    3、render
    4、conponentDidUpdate
    ```
    其中有一个shouldUpdateComponent方法，默认是return true，也即一定会调用render重新渲染；但是有时候只是父组件的数据变化，子组件的prop和state都未变化，这样也会触发子组件重新render，这样就造成了浪费；那么就需要shouldUpdateComponent这个方法过滤掉不必要的更新；方法定义如下：
    ```
    shouldComponentUpdate: function(nextProps, nextState){
        return this.state.checked === nextState.checked;
        //return false 则不更新组件
    }
    ```
    参数中包含最新的props和state，这样就可以拿来与this.state、this.props中的数据进行比较，如果所有数据都没有更新，则直接return false；阻止render的调用；但是state和props都是复杂对象，对比两个负责对象是否一致如果没有好的算法，则会带来更多的开销，immutablejs给出了高效的解决方案；这就是为什么要用不可变数据的原因，为了可以比较负责对象；
    
4. immutablejs解决了哪些问题？
    
    a. 只要保证每次都返回一个新对象，那么我只要直接比较this.state与nextState(this.props与nextProps)的引用是否相同即可判断是否做了修改,无需深度遍历对象；
    
    b. 对于复杂对象，目前如果不是用es6，修改对象/数组，并返回新对象/数组并不容易；如果用es6， 则可以无需使用immutablejs；例如：
    ```
    要修改数组第3个位置的对象里面的元素，这样要保证返回新的数组， 并且第三个对象也要返回新的对象；
    ```
    
   最后记录下看到的问题： 

Q:Immutable.js vs Object.assign()

A:二者不冲突，ImmutableJs除了解决对象的修改问题， 还给出了array等各种结构数据的修改方法；

# 读ReactAPI
第一部分是本次学习的收获，下面就是详细介绍react官网中列出的一些API，一些API虽然很少使用， 但是可以帮助更好的理解React；

#### React.createClass， React.createElement，React.createFactory
```
ReactClass createClass(object specification)
ReactElement createElement(
  string/ReactClass type,
  [object props],
  [children ...]
)
factoryFunction createFactory(
  string/ReactClass type
)
```
demo:
```
var Car = React.createClass({
  displayName: "Car",

  render: function render() {
    return React.createElement(
      "h1",
      { className: "red" },
      "audi"
    );
  }
});

var Person = React.createClass({
  displayName: "Person",

  render: function render() {
    return React.createElement(Car, null);
  }
});

ReactDOM.render(React.createElement(Person, null), document.body);
```

#### 理解React.createFactory

为了简化 React.createElement 的调用语法，React.createFactory 被引入：
```
var div = React.createFactory('div');
var root = div({ className: 'my-div' });
React.render(root, document.getElementById('example'));

React.createFactory 的定义基本就是如下形式，Element 的类型被提前绑定了。
function createFactory(type) {
  return React.createElement.bind(null, type);
}
```
React.DOM.div 等都是预先定义好的 “Factory”。“Factory” 用于创建特定 “ReactClass” 的 “Element”。

备注：

偏函数（Partial Functions）
Partial Functions也叫Partial Applications，这里截取一段关于偏函数的定义：

Partial application can be described as taking a function that accepts some number of arguments, binding values to one or more of those arguments, and returning a new function that only accepts the remaining, un-bound arguments.
这是一个很好的特性，使用bind()我们设定函数的预定义参数，然后调用的时候传入其他参数即可：
```
function list() {  
  return arguments;
}

var list1 = list(1, 2, 3); // [1, 2, 3]

// 预定义参数37
var leadingThirtysevenList = list.bind(undefined, 37);

var list2 = leadingThirtysevenList(); // [37]  
var list3 = leadingThirtysevenList(1, 2, 3); // [37, 1, 2, 3]  
```

#### 什么是ReactClass、ReactElement
ReactClass: A model or a shape for components, if you want to create a component called DatePicker, the shape is the ReactClass, not each of the individual components/instances will you create based on that shape.

var MyComponent = React.createClass({
 render: function() {
    ...
  }
});
ReactComponent or ReactElement: A component instance that you create based on a pre-defined ReactClass. If you create a component called DatePicker, each instance of that component that is rendered is a ReactElement

var component = React.createElement(MyComponent, props);
Normally you don't use React.createElement because you can just use something like this <MyComponent />, but it's really useful if you want to dynamically create instances of components, like for storing in a variable and rendering them later.

#### React.unmountComponentAtNode
```
boolean unmountComponentAtNode(DOMElement container)
```
从 DOM 中移除已经挂载的 React 组件，清除相应的事件处理器和 state。如果在 container 内没有组件挂载，这个函数将什么都不做。如果组件成功移除，则返回 true；如果没有组件被移除，则返回 false。

#### React.renderToString
```
string renderToString(ReactElement element)
```
把组件渲染成原始的 HTML 字符串。该方法应该仅在服务器端使用。React 将会返回一个 HTML 字符串。你可以在服务器端用此方法生成 HTML，然后将这些标记发送给客户端，这样可以获得更快的页面加载速度，并且有利于搜索引擎抓取页面，方便做 SEO。

如果在一个节点上面调用 React.render()，并且该节点已经有了服务器渲染的标记，React 将会维护该节点，并且仅绑定事件处理器，保证有一个高效的首屏加载体验。

demo
```
var Car = React.createClass({
  render:function() {
    return <h1 className="red">audi</h1>
  }
});


var Person = React.createClass({
  render:function() {
    return <Car />;
  }
});

console.log(ReactDOMServer.renderToString(<Person />)); 
//<h1 class="red" data-reactroot="" data-reactid="1" data-react-checksum="1599083419">audi</h1>
console.log(ReactDOMServer.renderToStaticMarkup(<Person />));
//<h1 class="red">audi</h1>
```


#### React.renderToStaticMarkup
```
string renderToStaticMarkup(ReactElement element)
```
和 renderToString 类似，除了不创建额外的 DOM 属性，例如 data-react-id，因为这些属性仅在 React 内部使用。如果你想用 React 做一个简单的静态页面生成器，这是很有用的，因为丢掉额外的属性能够节省很多字节。


#### React.isValidElement
```
boolean isValidElement(* object)
```
判断对象是否是一个 ReactElement。作用不明，可能是给测试使用的；

#### React.DOM
React.DOM 运用 React.createElement 为 DOM 组件提供了方便的包装。该方式仅在未使用 JSX 的时候适用。例如，React.DOM.div(null, 'Hello World!')。

#### React.PropTypes
用于验证传入组件的 props；
demo:
```
React.createClass({
  propTypes: {
    // 可以声明 prop 为指定的 JS 基本类型。默认
    // 情况下，这些 prop 都是可传可不传的。
    optionalArray: React.PropTypes.array,
    optionalBool: React.PropTypes.bool,
    optionalFunc: React.PropTypes.func,
    optionalNumber: React.PropTypes.number,
    optionalObject: React.PropTypes.object,
    optionalString: React.PropTypes.string,

    // 所有可以被渲染的对象：数字，
    // 字符串，DOM 元素或包含这些类型的数组。
    optionalNode: React.PropTypes.node,

    // React 元素
    optionalElement: React.PropTypes.element,

    // 用 JS 的 instanceof 操作符声明 prop 为类的实例。
    optionalMessage: React.PropTypes.instanceOf(Message),

    // 用 enum 来限制 prop 只接受指定的值。
    optionalEnum: React.PropTypes.oneOf(['News', 'Photos']),

    // 指定的多个对象类型中的一个
    optionalUnion: React.PropTypes.oneOfType([
      React.PropTypes.string,
      React.PropTypes.number,
      React.PropTypes.instanceOf(Message)
    ]),

    // 指定类型组成的数组
    optionalArrayOf: React.PropTypes.arrayOf(React.PropTypes.number),

    // 指定类型的属性构成的对象
    optionalObjectOf: React.PropTypes.objectOf(React.PropTypes.number),

    // 特定形状参数的对象
    optionalObjectWithShape: React.PropTypes.shape({
      color: React.PropTypes.string,
      fontSize: React.PropTypes.number
    }),

    // 以后任意类型加上 `isRequired` 来使 prop 不可空。
    requiredFunc: React.PropTypes.func.isRequired,

    // 不可空的任意类型
    requiredAny: React.PropTypes.any.isRequired,

    // 自定义验证器。如果验证失败需要返回一个 Error 对象。不要直接
    // 使用 `console.warn` 或抛异常，因为这样 `oneOfType` 会失效。
    customProp: function(props, propName, componentName) {
      if (!/matchme/.test(props[propName])) {
        return new Error('Validation failed!');
      }
    }
  }
});
```


#### React.Children
```
React.Children 为处理 this.props.children

React.Children.map
object React.Children.map(object children, function fn [, object context])
```
这个封闭的数据结构提供了有用的工具。

demo:

```
<Form>
    <p>Personal Information</p>
    <Input name="first_name" />
    <Input name="last_name" />
    <Input name="email" />
    <Label>
        Enter Your Birthday
        <Input name="birthday" type="date" />
    </Label>
</Form>

var newChildren = React.Children.map(this.props.children, function(child) {
    if (is.inArray(child.type.displayName, supportedInputTypes)) {
      var extraChildProps = {
        alertColor: this.props.alertColor,
        displayErrors: this.state.displayErrors
      }
      return React.cloneElement(child, extraChildProps);
    } else {
      return child;
    }
  }.bind(this));
```

备注：

1. 为什么不是this.props.children.map， 而需要React.Children.map?
因为this.props.children不是一个数组，而是一个对象；如下：
```
var Car = React.createClass({
  propTypes:{
    test:React.PropTypes.any.isRequired
  },
  render:function() {
    console.log(this.props.children);
    //Object {$$typeof: Symbol(react.element), type: "h1", key: null, ref: null, props: Object…}
    return <h1 className="red">{this.props.children}</h1>
  }
});


var Person = React.createClass({
  propTypes:{
    test:React.PropTypes.any.isRequired
  },
  render:function() {
    return <Car>
        <h1>123</h1>
    </Car>;
  }
});

ReactDOM.render(<Person />, document.body);
```

2. React.Children只获取第一级子元素， 不会深度遍历；


#### 组件定义和生命周期https://segmentfault.com/a/1190000004168886
组件在客户端被实例化，第一次创建时，以下方法将被调用：
```
getDefaultProps
getInitialState
componnetWillMount
render
componentDidMount
```

##### getDefaultProps :
用于设置组件props的默认值， 在组件内部不能修改props的值，因为props是外部传入的， 外部可能有多个同样的子组件，它们的props是共享的，如果一个变了， 所有组件都会被修改,组件的 props的属性可以通过父组件来更改;可以通过propTypes验证输入的props的正确性；如果不正确， 会通过console输出一个warning

##### getInitialState
用于初始化组件state的值；在这个方法里可以访问props，每个组件都有自己的state；
getInitialState 和 getDefaultPops 的调用是有区别的，getDefaultPops 是对于组件类来说只调用一次，后续该类的应用都不会被调用，而 getInitialState 是对于每个组件实例来讲都会调用，并且只调一次。

每次修改 state，都会重新渲染组件，实例化后通过 state 更新组件，会依次调用下列方法：
```
1、shouldComponentUpdate
2、conponentWillUpdate
3、render
4、conponentDidUpdate
```

但是不要直接修改 this.state，要通过 this.setState 方法来修改。
```
NEVER mutate this.state directly, as calling setState() afterwards may replace the mutation you made. Treat this.state as if it were immutable.

setState() does not immediately mutate this.state but creates a pending state transition. Accessing this.state after calling this method can potentially return the existing value.

There is no guarantee of synchronous operation of calls to setState and calls may be batched for performance gains.  setState() will always trigger a re-render unless conditional rendering logic is implemented in shouldComponentUpdate().

If mutable objects are being used and the logic cannot be implemented in shouldComponentUpdate(), calling setState() only when the new state differs from the previous state will avoid unnecessary re-renders.
```
```
getInitState: function () {
  return {
    count: 1,
    user: {
      school: {
        address: 'beijing',
        level: 'middleSchool'
      }
    }
  }
},
handleChangeSchool: function () {
  this.setState({
    user: {
      school: {
        address: 'shanghai'
      }
    }
  })
}
render() {
  console.log(this.state.user.school);
  // {address: 'shanghai'}, level 丢失， 因为他是通过Object.assign进行的浅merge;
}
```

##### componentWillMount
该方法在首次渲染之前调用;

#### render

该方法会创建一个虚拟DOM，用来表示组件的输出。对于一个组件来讲，render方法是唯一一个必需的方法。render方法需要满足下面几点：
```
只能通过 this.props 和 this.state 访问数据（不能修改）
可以返回 null,false 或者任何React组件
只能出现一个顶级组件，不能返回一组元素
不能改变组件的状态
不能修改DOM的输出
```
render方法返回的结果并不是真正的DOM元素，而是一个虚拟的表现，类似于一个DOM tree的结构的对象。react之所以效率高，就是这个原因。

##### componentDidMount
该方法不会在服务端被渲染的过程中调用。该方法被调用时，已经渲染出真实的 DOM，可以再该方法中通过 this.getDOMNode() 访问到真实的 DOM(推荐使用 ReactDOM.findDOMNode())。
```
var data = [..];
var comp = React.createClass({
    render: function(){
        return <imput .. />
    },
    conponentDidMount: function(){
        $(this.getDOMNode()).autoComplete({
            src: data
        })
    }
})
```

##### componentWillReceiveProps
组件的 props 属性可以通过父组件来更改，这时，componentWillReceiveProps 将来被调用。可以在这个方法里更新 state,以触发 render 方法重新渲染组件。
```
componentWillReceiveProps: function(nextProps){
    if(nextProps.checked !== undefined){
        this.setState({
            checked: nextProps.checked
        })
    }
}
```

##### shouldComponentUpdate
如果你确定组件的 props 或者 state 的改变不需要重新渲染，可以通过在这个方法里通过返回 false 来阻止组件的重新渲染，返回 `false 则不会执行 render 以及后面的 componentWillUpdate，componentDidUpdate 方法。

```
shouldComponentUpdate: function(nextProps, nextState){
    return this.state.checked === nextState.checked;
    //return false 则不更新组件
}
```

##### componentWillUnmount
每当React使用完一个组件，这个组件必须从 DOM 中卸载后被销毁，此时 componentWillUnmout 会被执行，完成所有的清理和销毁工作，在 conponentDidMount 中添加的任务都需要再该方法中撤销，如创建的定时器或事件监听器。

当再次装载组件时，以下方法会被依次调用：
```
1、getInitialState
2、componentWillMount
3、render
4、componentDidMount
```
![](https://segmentfault.com/img/bVrFki)

#### statics
```
object statics
```
statics 对象允许你定义静态的方法，这些静态的方法可以在组件类上调用。例如：
```
var MyComponent = React.createClass({
  statics: {
    customMethod: function(foo) {
      return foo === 'bar';
    }
  },
  render: function() {
  }
});

MyComponent.customMethod('bar');  // true
```
在这个块儿里面定义的方法都是静态的，意味着你可以在任何组件实例创建之前调用它们，这些方法不能获取组件的 props 和 state。如果你想在静态方法中检查 props 的值，在调用处把 props 作为参数传入到静态方法。


##### 其他
1. this.state.xxx = xxx 不会触发render；所以要用this.setState，它会触发render；

2. 虽然React使用virtualdom已经实现了最低成本的DOM更新，但是每次调用this.setState都会触发render，render后及时子组件的state没变化， 子组件的render也会被调用；这样造成浪费；所以要使用生命周期中的shouldComponentUpdate，如果子组件state没变化就return false； 不要更新；这里就引入了如果判断是否更新，详细参考：最上面Ref中的react如何性能达到最大化(前传)，暨react为啥非得使用immutable.js；


3. 更新列表item状态

```
class Person extends React.Component {
  
  constructor(props, context) {
    super(props, context);
    this.state = {
      list:[{id:1,active:1},{id:2,active:2}]
    };
  }
  
  componentWillReceiveProps(newProps){
    console.log(`我新的props的name是`);
  }

  handleClick(crt) {
    this.setState({
      list:this.state.list.map((item)=>item.id == crt.id ? {...item, active:3} : item)
    })
  }
  render() {
    var list = this.state.list;
    console.log(list);
    return (
        <div>
        {
          list.map((item) => <button onClick={()=>this.handleClick(item)}>test{item.active}</button>)
        }
        </div>
     )
  }
} 

ReactDOM.render(<Person/>, document.body);
```


# Refs:
1. [怎么更好的理解虚拟DOM?](https://www.zhihu.com/question/29504639)
2. [深度剖析：如何实现一个 Virtual DOM 算法 ](https://github.com/livoras/blog/issues/13)
3. [react如何性能达到最大化(前传)，暨react为啥非得使用immutable.js](https://segmentfault.com/a/1190000004290333)
4. [React 源码剖析系列 － 不可思议的 react diff](https://segmentfault.com/a/1190000004003055)
5. [React Mixin 的前世今生](https://segmentfault.com/a/1190000004034031)
6. [Immutable 详解及 React 中实践](https://github.com/camsong/blog/issues/3)