# 展示组件和容器组件分离在React项目中的实践

React提供了强大的组件复用系统，组件设计也非常灵活，几乎没有什么约束。但强大的灵活性，在实际开发中，也带来了问题。可能在同一项目中，不同开发者设计出来的组件就有不同的形态，导致组件不好复用，代码也不好维护。因此，我们需要有一套可行的原则来规范组件的设计。

在React技术社区，有一种广泛讨论的模式，那便是在设计时将组件分为展示组件和容器组件。但实际应用到项目中，对这一原则，不同人有不同理解。下面，结合个人在项目中的实践，谈谈对这一模式的理解。

## 展示组件
它是组件可复用的基本单元，与外部业务逻辑解耦，仅关心UI展现。一个展示组件应该符合以下特征：

* 主要关注UI展示
* 通过props参数控制业务逻辑，内部仅存在与UI相关的state
* 组件拥有自己的CSS样式，便于复用和移植
* 可以包含this.props.children元素
* 可以包含其他展示组件

## 容器组件
它除了是展示组件和其他容器组件的容器，还承载了应用的业务逻辑，负责与服务器打交道以及各种数据的加工清洗等。一个容器组件应该符合以下特征：

* 主要关注业务逻辑和数据加工
* 至少包含一个展示组件或容器组件
* 为展示组件或其他组件提供数据和方法
* 无自己的CSS样式，基本不包含DOM标签
* 通常通过state或回调函数控制应用业务逻辑

## 二者关系
![image](/assets/img/component.png)

## 实践中的一些额外约束
* 展示组件不能包含任何容器组件
* 展示组件可以包含展示组件，但组件设计尽量扁平化，避免展示组件的多层嵌套
* 通过props传递给下级组件的数据（包括当前组件state数据，上一级的props等）尽量扁平化，尽量避免一个对象里包含结构复杂的数据

## 看个简单的例子

我们先来定义一个简单的组件，然后重构该组件，将其分成容器组件和展示组件。例子中的组件从服务端获取数据并显示到页面上。

```js
// PageView.js
class PageView extends React.Component {
  constructor() {
    super();
    this.state = {
      pv: 0
    };
    this.getPV = this.getPV.bind(this);
  }
  
  componentDidMount() {
    this.getPV();
  }
  
  getPV() {
   ajax({
      url: "/pv.json",
      dataType: 'json'
    }).then((pv) => {
      this.setState({pv: pv});
    });
  }
  
  render() {
    return <div> 当前PV值是：{this.state.pv} </div>;
  }
}

```

上面的组件：
* 复用性 组件数据与视图没有分离，组件不能复用
* 可维护性 组件既承担了应用的业务逻辑部分，又承担了视图显示部分，承担的职责太多，如果业务稍微复杂，组件将变得难以维护

下面，我们根据展示组件和容器组件分离原则来重新设计它：

```js
// PageViewContainer.js - 容器组件
class PageViewContainer extends React.Component {
  constructor() {
    super();
    this.state = {
          pv: 0
        };
    this.getPV = this.getPV.bind(this);
  }
  
  componentDidMount() {
     this.getPV();
   }
   
   getPV() {
    ajax({
       url: "/pv.json",
       dataType: 'json'
     }).then((pv) => {
       this.setState({pv: pv});
     });
   }
   
  render() {
    return <PageView title="当前PV值是" count={this.state.pv} />;
  }
}


// PageView.js - 展示组件
class PageView extends React.Component {
  static propTypes = {
    title: PropTypes.string.isRequired,
    count: PropTypes.number.isRequired
  };
    
  render() {
    let {title, count} = this.props;
    return <div> {title}：{count} </div>;
  }
}
```

分离后的组件虽然代码量略增，但职责更加清晰，后续维护也更容易，而且PageView也可以复用了。

## 好处
  总结起来，组件分离带来的好处，有以下几点：
  
* 分离关注点使代码逻辑结构更清晰，更易维护。随着功能不断迭代，代码不断膨胀，清晰的逻辑划分，使问题更容易被定位，解耦模块间的依赖，修改BUG时也不易引入新的问题
* 展示组件和容器组件的分离使组件更容易在不同项目间复用
* 不改动任何业务逻辑代码的情况下，仅替换UI展示成为可能。也就是说通过组件业务逻辑与UI视图的分离，允许你可以任意组合你的容器组件和展示组件。

当然，这种分离并非必须遵循的原则，特别是在做业务开发的过程中，有时候很难界定一个组件是容器组件还是展示组件，怎么做，只能根据业务场景灵活应对了。
