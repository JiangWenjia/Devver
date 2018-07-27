入门四个小例子，先感受下React

1.Hello World!
```js
const element = <h1>Hello World!</h1>

ReactDOM.render(
    element,
document.getElementById('root')
);
```
2.函数

```js
function tick() {
    const element = (
        <div>
            <h1>Hello React</h1>
            <h2>现在是{new Date().toLocaleDateString()}</h2>
        </div>
    );

    ReactDOM.render(
        element,
        document.getElementById('root')
    )
}

setInterval(tick,1000);
```

3.属性

```js
function Clock(props) {
    return (
        <div>
            <h1>time is {props.date.toLocaleString()}</h1>
        </div>
    );
}
function tick() {
    ReactDOM.render(
        <Clock date={new Date()}/>,document.getElementById('root')
    );
}
setInterval(tick,1000);
```
4.类

```js
class Clock extends React.Component {
    render() {
        return (
          <div>
              <h1>TIME: {this.props.date.toLocaleString()}</h1>
          </div>
        );
    }
}
function tick() {
    ReactDOM.render(
      <Clock date={new Date()}/>,
        document.getElementById('root')
    );
}
setInterval(tick,1000);
```

# JSX
React使用JSX替代JavaScript，看起来像 XML JavaScript

## 表达式
表达式使用{},括起来。

```js
var root = document.getElementById('root');

ReactDOM.render(
  <div>
      <h1>{1+1}</h1>
  </div> ,
    root
);
```

## 样式

```js
var root = document.getElementById('root');
var myStyle = {
    fontSize: 100,
    color: '#878788'
}
ReactDOM.render(
    // 我是注释
    /*我也是注释*/
    <h2 style={myStyle}>XXXX
    </h2>,
    root
)
```
## 数组
```js
var root = document.getElementById('root');
var arr = [
    <h1>1</h1>,
    <h2>2</h2>,
];
ReactDOM.render(
    <div>{arr}</div>,
    root
)
```

# 组件
组件中可以复合嵌套使用

- 函数定义组件

```js
function HelloMessage(props) {
    return <h1>Hello World!</h1>;
}
```

- ES6 class定义组件

```js
var root = document.getElementById('root')

class Welcome extends React.Component {
    render() {
        return <h1>Hello World</h1>;
    }
}

ReactDOM.render(<Welcome/>,root);
```


- 自定义组件`<HelloMessage name="-World">`

```js
var root = document.getElementById('root')

/*函数*/
function HelloMessage(props) {
    return <h1>Hello{props.name}</h1>
}
const element = <HelloMessage name="-World"/>;
ReactDOM.render(element,root);
```

#状态
React的组件可以看作是状态机，通过交互实现不同的状态，然后渲染UI。

```js
var root = document.getElementById('root')

class Clock extends React.Component {
    constructor(props) {
        super(props)
        this.state  = {date: new Date()};
    }

    render() {
        return (
          <div>
              <h1> time : {this.state.date.toLocaleDateString()}</h1>
          </div>
        );
    }
    /*组件被挂在到DOM*/
    componentDidMount() {
        this.timerID = setInterval(
            () => this.tick(),
            1000
        );
    }
    /*组件从DOM中移除*/
    componentWillUnmount() {
        clearInterval(this.timerID);
    }

    tick() {
        this.setState({date:new Date()});
    }
}

ReactDOM.render(<Clock/>,root);

```

# Props 
props和state的区别主要在于，props是不可变的，state可以交互改变

```js
var root = document.getElementById('root');
function HelloMessage(props) {
    return <h1> Hello {props.name} !</h1>
}
const element = <HelloMessage name="World"/>
ReactDOM.render(<Clock/>,root);
```

### 默认Props

```js
var root = document.getElementById('root');

class HelloMessage extends React.Component {
    render() {
        return (
          <h1> Hello, {this.props.name} !</h1>
        );
    }
}

/*默认的props*/
HelloMessage.defaultProps = {
    name : 'World'
}

const element = <HelloMessage name="World"/>

ReactDOM.render(<HelloMessage/>,root);
```


# 事件
阻止事件的默认行为` e.preventDefault();`
```js
var root = document.getElementById('root');

function ActionLink() {
    function handleClick(e) {
        e.preventDefault();
        console.log('点击了链接');
    }
    return (
        <a  href="#" onClick={handleClick}> href</a>
    );
}
ReactDOM.render(<ActionLink/>,root);
```

例子2

```js
class Toggle extends React.Component {
    constructor(props) {
        super(props);
        this.state = {isToggleOn : true};
        this.handleClick = this.handleClick.bind(this);
    }

    handleClick() {
        this.setState(
            prevState => ({
                isToggleOn: !prevState.isToggleOn
            })
        )
    }

    render() {
        return <button onClick={this.handleClick}>
            {this.state.isToggleOn ? 'ON' : 'OFF'}
        </button>
    }
}
ReactDOM.render(<Toggle />,root);
```

传递参数，使用绑定的方式，不过事件e，要放到最后

```js
class Popper extends React.Component {
    constructor(props){
        super(props);
        this.state = {name:'Hello world'}
    }

    preventPop(name,e) {
        e.preventDefault();
        alert(name);
    }

    render(){
        return(
          <div>
              <p>hello</p>
              {
                  <a href="http://www.baidu.com" onClick={this.preventPop.bind(this,this.state.name)}>Click</a>
              }
          </div>
        )
    }
}
```

# 条件渲染
1.  使用if else

```js
var root = document.getElementById('root');


function UserGreeting(props) {
    return <h1>Welcome back!</h1>;
}

function GuestGreenting(props)  {
    return <h1>Please sign up!</h1>
}


function Greeting(props) {
    const isLoggedIn = props.isLoggedIn;
    if(isLoggedIn) {
      return <UserGreeting/>
    }
    return <GuestGreenting/>
}

ReactDOM.render(
  <Greeting isLoggedIn={true}/> ,root
);
```

2.元素变量

```js

var root = document.getElementById('root');

function UserGreeting(props) {
    return <h1>Welcome back!</h1>;
}

function GuestGreeting(props) {
    return <h1>Please sign up.</h1>;
}

function Greeting(props) {
    const isLoggedIn = props.isLoggedIn;
    if (isLoggedIn) {
        return <UserGreeting />;
    }
    return <GuestGreeting />;
}

function LoginButton(props) {
    return (
        <button onClick={props.onClick}>
            Login
        </button>
    );
}

function LogoutButton(props) {
    return (
        <button onClick={props.onClick}>
            Logout
        </button>
    );
}

class LoginControl extends React.Component {
    constructor(props) {
        super(props);
        this.handelLogoutClick = this.handelLogoutClick.bind(this);
        this.handleLoginClick = this.handleLoginClick.bind(this);
        this.state = {isLoggedIn:false};
    }

    handleLoginClick() {
        this.setState({
            isLoggedIn: true
        });
    }

    handelLogoutClick() {
        this.setState(
            {
                isLoggedIn: false
            }
        );
    }

    render() {
        const isLoggedIn = this.state.isLoggedIn;
        let button = null;
        if(isLoggedIn) {
            button = <LogoutButton onClick = {this.handelLogoutClick}/>
        }else {
            button = <LoginButton onClick = {this.handleLoginClick}/>;
        }

        return (
            <div>
                <Greeting isLoggedIn={isLoggedIn}></Greeting>
                {button}
            </div>
        );
    }
};

ReactDOM.render(<LoginControl/>,root);

```

##  与运算符 &&

```js
var root = document.getElementById('root');

function MailBox(props) {
    const unreadMessage = props.unreadMessage;

    return (
      <div>
          <h1>Hello!</h1>
          {
              unreadMessage.length > 0 &&
                  <h2>
                     You have {unreadMessage.length} unread message.
                  </h2>
          }
      </div>
    );
}

const message = [];

ReactDOM.render(<MailBox unreadMessage={message}/>,root);
```
## 三目运算符

```js
function Test(props) {
    const isLoggedIn = props.isLoggedIn;

    return(
      <div>
          {
              isLoggedIn ? (
                  <button >退出登录</button>
              ) : (
                  <button >登录</button>
              )
          }
      </div>
    );
}
ReactDOM.render(<Test isLoggedIn={false}/>,root);
```

## 阻止渲染

返回 null 即可
```js

function WarningBanner(props) {
    if(!props.warn) {
        return null;
    }

    return (
      <div> Warning!</div>
    );
}

class Page extends React.Component {
    constructor(props) {
        super(props);
        this.state = {showWaring:true};
        this.handleToggleClick = this.handleToggleClick.bind(this);
    }

    handleToggleClick() {
        this.setState(
          prevState => ({
             showWaring: !prevState.showWaring
          })
        );
    }

    render() {
        return(
            <div>
                <WarningBanner warn={this.state.showWaring}/>
                <button onClick={this.handleToggleClick}>
                    {this.state.showWaring ? 'Hide' : 'Show'}
                </button>
            </div>
        )
    }
}



ReactDOM.render(<Page />,root);
```

# 列表 & keys

```js
const numbers = [1, 2, 3, 4, 5];

const doubledNums = numbers.map(
    (number) => number * 2
);

console.log(doubledNums);

const listItems = numbers.map(
    (number) =>
        <li>{number}</li>
)

ReactDOM.render(<ul>{listItems}</ul>,root);
```

key 需要在数组的上下文中绑定

```js
const posts = [
    {id: 1, title: 'Hello World', content: 'Welcome to learning React!'},
    {id: 2, title: 'Installation', content: 'You can install React from npm.'}
];

function Blog(props) {
    const sidebar = (
      <ul>
          {
              props.posts.map((post) =>
                <li key={post.id}>
                    {post.title}
                </li>

              )
          }
      </ul>
    );

    const content = props.posts.map((post) =>
        <div key={post.id}>
            <h3>{post.title}</h3>
             <p>{post.content}</p>
        </div>
    );

    return (
        <div>
            {sidebar}
            <hr/>
            {content}
        </div>
    )
}
```

# 表单
需要其可控制，value 为 this.state.value
触发由内部函数处理。

```js
class NameForm extends React.Component {
    constructor(props) {
        super(props);
        this.state = {value: ''};
        this.handleChange = this.handleChange.bind(this);
        this.handelSubmit = this.handelSubmit.bind(this);
    }

    handleChange(event) {
        this.setState({
            value: event.target.value.toUpperCase()
        });
    }

    handelSubmit(event) {
        alert("submit");
        event.preventDefault();
    }

    render() {
       return(<form onSubmit={this.handelSubmit}>
            <label>Name:
            <input type="text" value={this.state.value} onChange={this.handleChange}/>
            </label>
            <textarea value={this.state.value} onChange={this.handleChange}></textarea>
           {/*file标签是只读的非受控制的*/}
           <input type="file"/>
           <br/>
           <label >pickMe
            <select value={this.state.value} onChange={this.handleChange}>
                <option value="lime1">lime1</option>
                <option value="lime2">lime2</option>
                <option value="lime3">lime3</option>
            </select>
           </label>
            <input type="submit" value="submit"/>
        </form>)
    };
}
```

多个输入的，可以使用event.target.name来区分

```js
class Reservation extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            isGoing: true,
            numberOfGuests: 2,
        }
        this.handleInputChange = this.handleInputChange.bind(this);
    }

    handleInputChange(event) {
        const target = event.target;
        const value = target.type === 'checkbox' ? target.checked : target.value;
        const name = target.name;
        this.setState({
            [name]: value
        });
    }

    render() {
      return( <form>
            <label>
                Is going:
                <input name="isGoing" type='checkbox' checked={this.state.isGoing}
                onChange={this.handleInputChange}
                />
                <label>
                    Number of guests
                    <input
                        name="numberOfGuests"
                        type="number"
                        value={this.state.numberOfGuests}
                        onChange={this.handleInputChange}
                    />
                </label>
            </label>
        </form>
      )
    }
}
```