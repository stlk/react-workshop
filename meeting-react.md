## Meeting React

Let's start our adventure in [CodePen](http://codepen.io/stlk/pen/vywaQB).

```js
ReactDOM.render(
  React.createElement('h1', {className: 'heading'}, 'Some cool heading'),
  document.getElementById('container')
);
```

#### `ReactDOM.render()`

The ReactDOM.render() function takes two arguments:
  - ReactElement to render
  - DOM node we want to render into - "entry point"

#### `React.createElement()`

This function is the main building block of React. It requires two arguments:
- tag name string (such as `'div'` or `'span'`), or a React component type (a class or a function)
- properties (such as `input` type)

```js
React.createElement('div');
// -> <div></div>
React.createElement('input', { type: 'radio' });
// -> <input type="radio"></input>
React.createElement('input', { className: 'form-control', type: 'radio' });
// -> <input class="form-control" type="radio"></input>
```

This would be pretty useless without having a way to specify content, so there's a third parameters where you can specify children.

```js
React.createElement('h1', null, 'Some cool heading');
// -> <h1>Some cool heading</h1>
```

Of couse you can use another React component as child.

```js
React.createElement('div', { className: 'wrapper' },
  React.createElement('h1', null, 'Some cool heading')
)
```

this creates

```html
<div class="wrapper">
  <h1>Some cool heading</h1>
</div>
```
This `.wrapper` will probably set width of content. By reusing this element, we make sure our application is consistent since it has the same styling everywhere. This is what components are for.

## Components

Basic `ReactComponent` is just a function that returns `ReactElement`:

```
var Wrapper = function(props) {
  return React.createElement('div', { className: 'wrapper' });
}
```

We can then use it the same way as any HTML elements:

```
React.createElement(Wrapper);
// -> <div class="wrapper"></div>
```

This works fine. Let's try adding some content:

```
React.createElement(Wrapper, {}, 'React rocks!?');
// -> <div class="wrapper"></div>
```

That didn't work :( That's because children are hidden inside `props.children`. All we need to do is to use them somewhere.

```
var Wrapper = function(props) {
  return React.createElement('div', { className: 'wrapper' }, props.children);
}
```

```
React.createElement(Wrapper, {}, 'React rocks!?');
// -> <div class="wrapper">React rocks!?</div>
```

Great! Now we have component whose content we can easily use whenever we want.
