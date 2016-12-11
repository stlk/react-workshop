## JSX

As mentioned in the “Why React?” section, React has the virtual DOM to minimize rerendering when the application state changes. But what is application state and how do we manage it in React?

Any real world application will have state. State can be anything and everything, ranging from “this checkbox is checked” over “that modal is open” to “this data was fetched”.

As a simple example of state, let’s create a Counter component that counts how often we’ve clicked a button! Our Wrapper component above was written as a functional component. To create stateful components, we have to use a slightly different notation to create components – the class notation!

To create a stateful component, we create a new class that extends React.Component. (React.Component is a base we can build upon that React provides for us) We assign it a render method from which we return our ReactElements, not unlike the functional component:

```js
class Counter extends React.Component {
  render() {
    return (
      <p>This is the Counter component!</p>
    );
  }
}
```

We can then render this component just like the other components with ReactDOM.render:

```js
ReactDOM.render(
  <Counter />,
  document.getElementById('container')
);
```

Let’s make a separate Button component, which will take a prop called text. We’ll make this component a functional one again, since it won’t need to store any state:

```js
var Button = function(props) {
  return (
    <button>{ props.text }</button>
  );
}
```

Then we render our Button into our Counter with a text of Click me!:

```js
class Counter extends React.Component {
  render() {
    return (
      <div>
        <p>This is the Counter component!</p>
        <Button text="Click me!"/>
      </div>
    );
  }
}
```

Now let’s increase a number every time a user clicks on our Button by using an onClick handler:

```js
class Counter extends React.Component {
  render() {
    return (
      <div>
        <p>This is the Counter component!</p>
        <Button text="Click me!" onClick={function() { console.log('click!') }} />
      </div>
    );
  }
}
```

Here we need to differentiate between react components and real DOM nodes. Event handlers, like onClick, onMouseOver, etc., only work when they are attached to a real DOM node. The above example doesn’t work, because we’re only attaching it to a ReactComponent. You can click the Button however much you like, you will never see "click!" in the console!

To make this work, we have to attach the onCLick handler to the native DOM button node inside the Button component:

```js
var Button = function(props) {
  return (
    <button onClick={props.onClick}>{ props.text }</button>
  );
}
```

Yey, this works!

Now let's do something userful in the `onClick` handler. State is a plain object in react, which can have as little or as many properties as you like! Out state will have a clicks property, which initially is zero and increments by one with each click.

The first thing we need to do is set the initial state. Classes have a constructor that is called when the class is first initialised, which we can use to assign the initial state to our component:

```js
class Counter extends React.Component {
  constructor() {
    super();
    this.state = {
      clicks: 0
    };
  }

  render() { /* ... */ }
}
```

That alone won’t do anything though, we don’t see that number anywhere on the page! To access the current state of our component anywhere within our component we access this.state. Let’s render the current number of clicks as text for a start:

```js
class Counter extends React.Component {
  constructor() {
    super();
    this.state = {
      clicks: 0
    };
  }

  render() {
    return (
      <div>
        <p>This is the Counter component! The button was clicked { this.state.clicks } times.</p>
        <Button text="Click me!" onClick={function() { console.log('click!') }} />
      </div>
    );
  }
}
```

To change the state of a component, we use the this.setState helper function which React provides. Let’s add an increment method to our Counter, which increments the clicks state by one, and call this.increment when our Button is clicked!

```js
class Counter extends React.Component {
  constructor() {
    super();
    this.state = {
      clicks: 0
    };
  }

  increment() {
    this.setState({
      clicks: this.state.clicks + 1
    });
  };

  render() {
    return (
      <div>
        <p>This is the Counter component! The button was clicked { this.state.clicks } times.</p>
        <Button text="Click me!" onClick={this.increment} />
      </div>
    );
  }
}
```

The problem here is that this is undefined in increment because of the way ES6 classes work – the easiest way to fix this is to bind the context of increment to the class in the constructor like so:

```js
class Counter extends React.Component {
  constructor() {
    super();
    this.state = {
      clicks: 0
    };
    // this.increment = this.increment.bind(this);
  }

  increment = () => {
    this.setState({
      clicks: this.state.clicks + 1
    });
  };

  render() {
    return (
      <div>
        <p>This is the Counter component! The button was clicked { this.state.clicks } times.</p>
        <Button text="Click me!" onClick={this.increment} />
      </div>
    );
  }
}
```
