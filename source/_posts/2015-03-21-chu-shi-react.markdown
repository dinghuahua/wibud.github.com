---
layout: post
title: "初识react"
date: 2015-03-21 16:12:08 +0800
comments: true
categories: 前端
---
 
## 是什么
	
> React is a JavaScript library for creating user interfaces by Facebook and Instagram. Many people choose to think of React as the V in MVC.

> We built React to solve one problem: building large applications with data that changes over time. To do this, React uses two main ideas.
	
套用官网的一句话

> React可以被认为是MVC中得View层。它是用来构建数据不断变化的大型发杂应用的。

所以不要拿React同Angular等框架作比较了，它只是View层的框架，没有什么可比性~~

在React中，贯穿始终的就是`组件`（组件化也是web发展的方向），React组件可以简单的认为是HTML和JavaScript的组合，通常的View层是使用一些模板工具生成，而React是通过组件的组合产出View，每个组件都可以独立或者配合使用（组件定义良好的话），其中React提供了一系列创建组件、渲染组件的方法，同时也对事件、状态等进行了封装。

为什么说React适合数据频繁变化的应用呢？

+ 数据变化时，只需更新组件的状态，剩下的操作React都帮你做了
+ React创造性的虚拟DOM和diff算法，使得其比传统的DOM更新更加高效，性能也更高

这里不得不提一下[JSX](http://facebook.github.io/jsx/)，它是和React配合使用的，当然不用也可以的（follow your heart~）。

> JSX 是一个看起来很像 XML 的 JavaScript 语法扩展。React 可以用来做简单的 JSX 句法转换。

简单说来，JSX就是可以将JavaScript 语法写成XML/HTML的那个样子，举个简单的例子：

	// 输入 (JSX):
	var app = <Nav color="blue" />;
	// JSX 把类 XML 的语法转成原生 JavaScript，XML 元素、属性和子节点被转换成 React.createElement 的参数。
	// 经JSX转化，输出 (JS):
	var app = React.createElement(Nav, {color:"blue"});


## 为什么

我是否该使用React？别急，下面我们来详细看下。

### 优点

+ 只需要声明怎么展示，数据变化时，React自动帮你处理，不需要关心怎么更新DOM
+ React唯一需要做的就是构建组件，组件良好的封装性，更易于代码复用和关注分离
+ 只需要查看组件的返回就能知道组件是如何渲染的，而不需要知道HTML和JS处理逻辑
+ 方便维护和扩展

### 缺点

+ 学习成本虽然不高，但是要理解和熟练掌握还是要花一定的时间，主要是官方文档错综发杂
+ React很年轻，社区也在发展中，相应的组件库并不丰富
+ React只是View层，实现一些复杂应用可能需要其他框架的支持

## 怎么用

### 如何显示

先来看一个官网上的简单例子：

	<!DOCTYPE html>
	<html>
		<head>
			<title>Hello React</title>
			<script src="http://fb.me/react-0.13.0.js"></script>
      		<script src="http://fb.me/JSXTransformer-0.13.0.js"></script>
	  	</head>
	  	<body>
      		<div id="example"></div>
			<script type="text/jsx">

				var HelloWorld = React.createClass({

					render: function() {
						return (
							<p>
								Hello, <input type="text" placeholder="Your name here" />!
								It is {this.props.date.toTimeString()}
							</p>
						);
					}
				});

				setInterval(function() {
					React.render(
						<HelloWorld date={new Date()} />,
						document.getElementById('example')
					);
				}, 500);

			</script>
		</body>
	</html>

在浏览器中运行这个例子，时间是每隔半秒钟变化一次。在输入框输入内容，你会发现 React 在用户界面中只改变了时间， 任何你在输入框输入的内容一直保留着，React知道哪些东西改变了，知道要更新哪些东西，并且帮你做好了。

其中用到了JSX，使用JSX的话，需要用`<script type="text/jsx">`标签包裹JSX代码。

对这个组件的输入称为 props - "properties"的缩写。得益于 JSX 语法，它们通过参数传递。不要去修改props的值， this.props 是只读的。

React 组件非常简单。你可以认为它们就是简单的函数，接受 props 和 state (后面会讨论) 作为参数，然后渲染出 HTML。正是应为它们是这么的简单，这使得它们非常容易理解。

### 如何交互

继续上官网里例子：

	var LikeButton = React.createClass({
		getInitialState: function() {
			return {liked: false};
		},
		handleClick: function(event) {
			this.setState({liked: !this.state.liked});
		},
		render: function() {
			var text = this.state.liked ? 'like' : 'haven\'t liked';
			return (
				<p onClick={this.handleClick}>
					You {text} this. Click to toggle.
				</p>
			);
		}
	});

	React.render(
		<LikeButton />,
		document.getElementById('example')
	);
	
在浏览器中运行这个例子，每次点击按钮，我们的文案就换了。这里我们绑定了事件，同时用到了state。

这里的事件绑定与处理是经过React封装过得，当然封装是透明的，用起来和原生事件一样。

可以把React组件想象成一个状态机，状态（state）改变时，就会重新渲染变化，让组件呈现的界面同数据保持一致，实现起来十分简单，只需要调用`setState`方法，改变this.state，然后React会帮我们根据新的状态来重新渲染（完全不用开发者操心）

#### 使用`setState`的几点注意：

+ `setState`不会立即更新state，也不会立即调用render进行重新渲染，看如下的例子

		var i = 0;

		var Counter = React.createClass({

			getInitialState: function(){

				return {count: i};
			},
			handle: function(e){

				i++;
				this.setState({count: i});
				console.log(this.state.count, 'click');
			},
			render: function() {
				console.log(this.state.count, 'render');
				return (
					<p onClick={this.handle}>
						i = {this.state.count}
					</p>
				);
			}
		});

		React.render(
			<Counter />,
			document.getElementById('example')
		);
		
	浏览器运行这个例子，页面渲染完，控制台会打印出`0 "render"`，点击一下<p>标签，打印出`0 "click"` `1 "render"`。自己想一下，不然可以得出结论。
	
+ `setState`是将参数合并（merge）到this.state上，而不是替换

#### 使用state的几点注意：

+ state只属于当前组件，不可传递
+ state应该只包含那些可能改变并触发用户界面更新的数据，其他的数据应该放到props中
+ 常用的模式是创建多个只负责渲染数据的无状态（stateless）组件，在它们的上层创建一个有状态（stateful）组件并把它的状态通过 props 传给子级。这个有状态的组件封装了所有用户的交互逻辑，而这些无状态组件则负责声明式地渲染数据。
+ state中只是存放一些元数据，更具其计算出的数据，不要放到state中，把计算放到render()中。

### 如何组合组件

React的很棒的特性之一就是他的组件的可组合性

	var User = React.createClass({
		render: function() {
			return (
				<div>
					<UserName name={this.props.name} />
				</div>
			);
		}
	});

	var UserName = React.createClass({
		render: function() {
			return (
				<p>
					{this.props.name}
				</p>
			);
		}
	});

	React.render(
		<User name="wibud" />,
		document.getElementById('example')
	);
	
也可以很简单的实现批量添加子组件：

	var User = React.createClass({
		render: function() {
			return (
				<div>
					{this.props.names.map(function(name){

						return <UserName name={name} />
					})}
				</div>
			);
		}
	});

	var UserName = React.createClass({
		render: function() {
			return (
				<p>
					{this.props.name}
				</p>
			);
		}
	});

	React.render(
		<User names={['wibud','wibud2','wibud3']} />,
		document.getElementById('example')
	);
	
父组件和子组件可以通过props来传递数据，其中父组件可以通过this.props.children来获取子组件。

值得注意的是：子组件会根据他们被渲染的顺序来更新DOM，如下两次渲染过程：

	// 第一次渲染
	<List>
		<p>item 1</p>
		<p>item 2</p>
	</List>
	// 第二次渲染
	<List>
		<p>item 2</p>
	</List>
	
直观来看，只是删除了`<p>item 1</p>`。事实上，React 先更新第一个子级的内容，然后删除最后一个组件。如果想要保证子组件渲染的正确顺序，即上述例子中删除`<p>item 1</p>`，只需要给子组件加上`key`参数即可：

	render: function() {
		var results = this.props.results;
		return (
			<ol>
				{results.map(function(result) {
					return <li key={result.id}>{result.text}</li>;
				})}
			</ol>
		);
	}
	
### 如何复用组件

#### propTypes

随着应用不断变大，保证组件被正确使用变得非常有用。为此我们引入 `propTypes	。`propTypes`是用来校验props数据的正确性的，如果数据类型不正确，会抛出异常。处于性能考虑，校验只在debug模式下有效。

具体可校验类型，参考[这里](http://facebook.github.io/react/docs/reusable-components.html)

使用起来也是超级的简单：

	var MyComponent = React.createClass({
		propTypes: {
			children: React.PropTypes.element.isRequired
		},

		render: function() {
			return (
				<div>
					{this.props.children} // 有且仅有一个元素，否则会抛异常。
				</div>
			);
		}
	});

#### Mixins

组件是 React 里复用代码最佳方式，但是有时一些复杂的组件间也需要共用一些功能。React 使用 mixins 来解决这类问题。

如下例子，我们实现了通用的setInterval的创建和销毁

	var SetIntervalMixin = {
		// 组件Render前调用
		componentWillMount: function() {
			this.intervals = [];
		},
		setInterval: function() {
			this.intervals.push(setInterval.apply(null, arguments));
		},
		// 组件销毁时调用
		componentWillUnmount: function() {
			this.intervals.map(clearInterval);
		}
	};

	var TickTock = React.createClass({
		// 引用 mixin
		mixins: [SetIntervalMixin],

		getInitialState: function() {
			return {seconds: 0};
		},
		// 组件render完后调用
		componentDidMount: function() {
			this.setInterval(this.tick, 1000); // 调用 mixin 的方法
		},
		tick: function() {
			this.setState({seconds: this.state.seconds + 1});
		},
		render: function() {
			return (
				<p>
					React has been running for {this.state.seconds} seconds.
				</p>
			);
		}
	});

	React.render(
		<TickTock />,
		document.getElementById('example')
	);
	
mixin相当于把通过mixin定义的方法属性赋到组件对象中，如果有多个mixin时，并用有多个 mixin 定义了同样的生命周期方法，即componentWillUnmount等方法（如：多个 mixin 都需要在组件销毁时做资源清理操作），所有这些生命周期方法都保证会被执行到。方法执行顺序是：首先按 mixin 引入顺序执行 mixin 里方法，最后执行组件内定义的方法。

### 组件生命周期

#### 组件初始化

+ 1、getDefaultProps()
+ 2、getInitialState()
+ 3、componentWillMount()
+ 4、render()
+ 5、componentDidMount()

#### 组件销毁

+ 1、componentWillUnmount()

#### 调用setState()后的更新流程

+ 1、shouldComponentUpdate()
+ 2、componentWillUpdate()
+ 3、render()
+ 4、componentDidUpdate()

### class和style

这里是针对JSX的

+ class处理

给节点设置样式时，需要用`className`，可以使用`React.addons.classSet`来设置多个class：

	render: function(){
		var classes = React.addons.classSet({
			'classOne': true,	// 需要
			'classTwo': false // 不需要
		});
		
		return <div className={classes}></div>
	}
	
渲染的结果里，div元素会只有classOne这一个样式类。

+ 内联style

实例如下：

	render: function(){
		var styles = {
			color: 'red',
			height: 10
		};
		
		return <div style={styles}></div>
	}

### 常用API

+ render()
	+ 渲染组件
	+ render方法要保持简单
	
+ componentWillMount()
	+ render前调用，只触发一次
	+ 可做一些初始化的工作
	
+ componentWillUnmount()
	+ 组件销毁前调用
	+ 做一些清理工作

+ getInitialState()
	+ 设置初始的state值
	
+ getDefaultProps()
	+ 设置默认的prop值
	+ 用户未设置时，会去默认值
+ setState()
	+ 更新state
	
更多API参见[官网](http://facebook.github.io/react/)

### Virtual DOM 和 Diff算法

有兴趣看下面的资料汇总

# 资料汇总

+ [官网](http://facebook.github.io/react/)
+ [文档教程(有一部分翻译成了中文)](https://github.com/facebook/react/tree/master/docs/docs)
+ [Complementary Tools工具和组件](https://github.com/facebook/react/wiki/Complementary-Tools)
+ react virtual dom
	+ [The Secrets of React's Virtual DOM](http://fluentconf.com/fluent2014/public/schedule/detail/32395)
	+ [Why is React's concept of Virtual DOM said to be more performant than dirty model checking?](http://stackoverflow.com/questions/21109361/why-is-reacts-concept-of-virtual-dom-said-to-be-more-performant-than-dirty-mode)
+ [react diff algorithm](http://calendar.perfplanet.com/2013/diff/)
+ [react-canvas](https://github.com/flipboard/react-canvas)
	+ [相关阅读](http://www.ruanyifeng.com/blog/2015/02/future-of-dom.html)
+ [React: Flux Architecture](https://egghead.io/series/react-flux-architecture)
+ [example of how to do server-side rendering with the React library](https://github.com/mhart/react-server-example)
+ [React 中文微博](http://weibo.com/reactchina)
+ [React 中文社区](http://react-china.org/)
+ [react-vs-angularjs](https://www.codementor.io/reactjs/tutorial/react-vs-angularjs)