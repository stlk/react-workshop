## JSX

You might have noticed that most React code uses it's own "templating engine" called JSX. It makes code more familiar and readable, but it's just a wrapper for React.createElement.

Instead of calling `React.createElement` manually, we can use JSX to make the code look more like the rendered HTML:

```jsx
<Wrapper>
  <h1 className="heading">React rocks!?</h1>
</Wrapper>
```

is the same as:

```js
React.createElement(Wrapper, null,
  React.createElement('h1', {className: 'heading'}, 'React rocks!?')
)
```

We can try it out on page https://jsx-live.now.sh.

> Since JSX is syntax is not supported by browsers, there needs to be something that converts this code back to JS. Codepen already took care of this, so we can go ahead and try it.

Let’s convert our `Wrapper` component to use JSX:

```jsx
var Wrapper = function(props) {
  return (
    <div className="wrapper">{ props.children }</div>
  );
};
```
