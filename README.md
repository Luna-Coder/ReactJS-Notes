# ReactJS-Notes
___
## Intro To JSX

JSX produces React “elements”.

```jsx
const element = <h1>Hello, world!</h1>;
```

### JSX is an Expression Too
After compilation, JSX expressions become regular JavaScript function calls and evaluate to JavaScript objects.

This means that you can use JSX inside of `if` statements and `for` loops, assign it to variables, accept it as arguments, and return it from functions.

```jsx
function getGreeting(user) {
  if (user) {
    return <h1>Hello, {formatName(user)}!</h1>;
  }
  
  return <h1>Hello, Stranger.</h1>;
}
```

### Specifying Attributes with JSX
You may use quotes, `" "`, to specify string literals as attributes.
```jsx
const element = <div tabIndex="0"></div>;
```
You may also use curly braces, `{ }`, to embed a JavaScript expression in an attribute.
```jsx
const element = <img src={user.avatarUrl}></img>;
```

### Specifying Children with JSX
If a tag is empty, you may close it immediately with `/>`, like XML.
```jsx
const element = <img src={user.avatarUrl} />;
```
JSX tags may contain children.
```jsx
const element = (
  <div>
    <h1>Hello!</h1>
    <h2>Good to see you here.</h2>
  </div>
);
```

### JSX Represents Objects
Babel compiles JSX down to `React.createElement()` calls.

The following two code snippets are identical under the hood.
```jsx
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```

```jsx
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
`React.createElement()` creates objects called "_React elements_".
```jsx
const element = {
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
};
```
Think of these objects like descriptions of what you want to see on the screen. 

React reads these objects and uses them to construct the DOM and keep it up to date.

___

## Rendering Elements

Elements are the smallest building blocks of React apps.

An element describes what you want to see on the screen
```jsx
const element = <h1>Hello, world</h1>;
```
Unlike browser DOM elements, React elements are plain objects, and are cheap to create. 

### Rendering an Element into the DOM
Let’s say there is a `<div>` somewhere in your HTML file.
```html
<div id="root"></div>
```
To render a React element into a root DOM node, pass both to `ReactDOM.render()`.
```jsx
const element = <h1>Hello, world</h1>;
ReactDOM.render(element, document.getElementById('root'));
```

### Updating the Rendered Element
React elements are `immutable`. Once you create an element, you can’t change its children or attributes.

With our knowledge so far, the only way to update the UI is to create a new element, and pass it to `ReactDOM.render()`.

Consider this ticking clock example.
```jsx
function tick() {
  const element = (
    <div>
      <h1>Hello, world!</h1>
      <h2>It is {new Date().toLocaleTimeString()}.</h2>
    </div>
  );
  ReactDOM.render(element, document.getElementById('root'));
}

setInterval(tick, 1000);    // It calls ReactDOM.render() every second from a setInterval() callback.
```
React DOM compares the element and its children to the previous one, and only applies the DOM updates necessary to bring the DOM to the desired state.

___

## Components and Props

Conceptually, components are like JavaScript functions. They accept arbitrary inputs (called `props`) and return React elements describing what should appear on the screen.

### Function and Class Components
```jsx
// Function Component
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}
```
This function is a valid React component because it accepts a single `props` object argument with data and returns a React element.

```jsx
// Class Component
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

### Rendering a Component
React elements can also represent user-defined components.
```jsx
const element = <Welcome name="Sara" />;
```
When React sees an element representing a user-defined component, it passes JSX attributes to this component as a single object. We call this object `props`.

For example, this code renders “Hello, Sara” on the page.
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

const element = <Welcome name="Sara" />;
ReactDOM.render(
  element,
  document.getElementById('root')
);
```
Let’s recap what happens in this example:

1. We call `ReactDOM.render()` with the `<Welcome name="Sara" />` element.

2. React calls the `Welcome` component with `{name: 'Sara'}` as the `props`.

3. Our `Welcome` component returns a `<h1>Hello, Sara</h1>` element as the result.

4. React DOM efficiently updates the DOM to match `<h1>Hello, Sara</h1>`.

### Composing Components
Components can refer to other components in their output. This lets us use the same component abstraction for any level of detail.

For example, we can create an App component that renders Welcome many times.
```jsx
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

function App() {
  return (
    <div>
      <Welcome name="Sara" />
      <Welcome name="Cahal" />
      <Welcome name="Edite" />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

### Props are Read-Only
Whether you declare a component as a function or a class, it must never modify its own props.

**All React components must act like pure functions with respect to their props.**

___

## State and Lifecycle

State is similar to `props`, but it is private and fully controlled by the component.

### Converting a Function to a Class
You can convert a function component like `Clock` to a class in five steps:

1. Create an ES6 class, with the same name, that extends `React.Component`.

2. Add a single empty method to it called `render()`.

3. Move the body of the function into the `render()` method.

4. Replace `props` with `this.props` in the `render()` body.

5. Delete the remaining empty function declaration.
```jsx
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.props.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
`Clock` is now defined as a class rather than a function.

The `render()` method will be called each time an update happens, but as long as we render `<Clock />` into the same DOM node, only a single instance of the `Clock` class will be used. 

This lets us use additional features such as local state and lifecycle methods.

### Adding Local State to a Class
We will move the `date` from props to state in three steps:

**1.** Replace `this.props.date` with `this.state.date` in the `render()` method.
```jsx
class Clock extends React.Component {
  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
**2.** Add a class constructor that assigns the initial `this.state`.
```jsx
class Clock extends React.Component {   // Note how we pass props to the base constructor.
  constructor(props) {            // Class components should always call the base constructor with props.
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}
```
**3.** Remove the `date` prop from the `<Clock />` element.
```jsx
ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
We will later add the timer code back to the component itself.

The result looks like this:
```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
Next, we’ll make the Clock set up its own timer and update itself every second.

### Adding Lifecycle Methods to a Class
In applications with many components, it’s very important to free up resources taken by the components when they are destroyed.

We want to set up a timer whenever the Clock is rendered to the DOM for the first time. This is called “mounting” in React.

We also want to clear that timer whenever the DOM produced by the Clock is removed. This is called “unmounting” in React.

We can declare special methods on the component class to run some code when a component mounts and unmounts. These are called "lifecycle methods".
```jsx
componentDidMount() {
    this.timerID = setInterval(     // Note how we save the timer ID right on this
      () => this.tick(),
      1000
    );
}
```
The `componentDidMount()` method runs after the component output has been rendered to the DOM. This is a good place to set up a timer.

While `this.props` is set up by React itself and `this.state` has a special meaning, you are free to add additional fields to the class manually if you need to store something that doesn’t participate in the data flow (like a timer ID).

```jsx  
componentWillUnmount() {
    clearInterval(this.timerID);
}
```
We will tear down the timer in the `componentWillUnmount()` lifecycle method.

Finally, we will implement a method called `tick()` that the `Clock` component will run every second.

It will use `this.setState()` to schedule updates to the component local state.
```jsx
tick() {
    this.setState({
      date: new Date()
    });
}
```
Here's the final result:
```jsx
class Clock extends React.Component {
  constructor(props) {
    super(props);
    this.state = {date: new Date()};
  }

  componentDidMount() {
    this.timerID = setInterval(
      () => this.tick(),
      1000
    );
  }

  componentWillUnmount() {
    clearInterval(this.timerID);
  }

  tick() {
    this.setState({
      date: new Date()
    });
  }

  render() {
    return (
      <div>
        <h1>Hello, world!</h1>
        <h2>It is {this.state.date.toLocaleTimeString()}.</h2>
      </div>
    );
  }
}

ReactDOM.render(
  <Clock />,
  document.getElementById('root')
);
```
Let’s quickly recap what’s going on and the order in which the methods are called:

**1.** When `<Clock />` is passed to `ReactDOM.render()`, React calls the constructor of the `Clock` component. Since `Clock` needs to display the current time, it initializes `this.state` with an object including the current time. We will later update this state.

**2.** React then calls the `Clock` component’s `render()` method. This is how React learns what should be displayed on the screen. React then updates the DOM to match the `Clock`’s render output.

**3.** When the `Clock` output is inserted in the DOM, React calls the `componentDidMount()` lifecycle method. Inside it, the `Clock` component asks the browser to set up a timer to call the component’s `tick()` method once a second.

**4.** Every second the browser calls the `tick()` method. Inside it, the `Clock` component schedules a UI update by calling `setState()` with an object containing the current time. Thanks to the setState() call, React knows the state has changed, and calls the `render()` method again to learn what should be on the screen. This time, `this.state.date` in the `render()` method will be different, and so the render output will include the updated time. React updates the DOM accordingly.

**5.** If the `Clock` component is ever removed from the DOM, React calls the `componentWillUnmount()` lifecycle method so the timer is stopped.

### Using State Correctly
There are three things you should know about `setState()`.

##### (1) Do Not Modify State Directly
For example, this will not re-render a component:
```jsx
// Wrong
this.state.comment = 'Hello';
```
Instead, use `setState()`:
```jsx
// Correct
this.setState({comment: 'Hello'});
```
The only place where you can assign `this.state` is the `constructor`.

##### (2) State Updates May Be Asynchronous
React may batch multiple setState() calls into a single update for performance.

Because this.props and this.state may be updated asynchronously, you should not rely on their values for calculating the next state.

For example, this code may fail to update the counter.
```jsx
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
```

To fix it, use a second form of `setState()` that accepts a function rather than an object. 

That function will receive the previous state as the first argument, and the props at the time the update is applied as the second argument.
```jsx
// Correct
this.setState((state, props) => ({
  counter: state.counter + props.increment
}));
```

##### (3) State Updates are Merged
When you call `setState()`, React merges the object you provide into the current state.

For example, your state may contain several independent variables.
```jsx
constructor(props) {
  super(props);
  this.state = {
    posts: [],
    comments: []
  };
}
 ```
Then you can update them independently with separate `setState()` calls:
```jsx
componentDidMount() {
  fetchPosts().then(response => {
    this.setState({
      posts: response.posts
    });
  });

  fetchComments().then(response => {
    this.setState({
      comments: response.comments
    });
   });
}
```
The merging is shallow, so `this.setState({comments})` leaves `this.state.posts` intact, but completely replaces `this.state.comments`.

### The Data Flows Down
Neither parent nor child components can know if a certain component is stateful or stateless.

This is why state is often called local or encapsulated. It is not accessible to any component other than the one that owns and sets it.

A component may choose to pass its state down as props to its child components:
```jsx
<h2>It is {this.state.date.toLocaleTimeString()}.</h2>
```
This also works for user-defined components:
```jsx
<FormattedDate date={this.state.date} />
```
The `FormattedDate` component would receive the `date` in its props and wouldn’t know whether it came from the `Clock`’s state, from the `Clock`’s props, or was typed by hand:
```jsx
function FormattedDate(props) {
  return <h2>It is {props.date.toLocaleTimeString()}.</h2>;
}
```
This is commonly called a “top-down” or “unidirectional” data flow. Any state is always owned by some specific component, and any data or UI derived from that state can only affect components “below” them in the tree.

If you imagine a component tree as a waterfall of props, each component’s state is like an additional water source that joins it at an arbitrary point but also flows down.

```jsx
function App() {
  return (
    <div>
      <Clock />       // All Clock components are truly isolated
      <Clock />       // Each Clock sets up its own timer and updates independently
      <Clock />
    </div>
  );
}

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```
In React apps, whether a component is stateful or stateless is considered an implementation detail of the component that may change over time. You can use stateless components inside stateful components, and vice versa.






