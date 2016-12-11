## Modules

Real world applications can have any number of components, ranging from a handful to thousands. Having all of them in a single file is impractical, so we structure them into modules. This allows us to keep our applications well structured and easy to work with. What we are doing above is telling the App component that we’re using the react module and the App.css module!

We can create modules by exporting something from a file, like above we’re exporting the App component. This component can then be imported in another file with import App from './path/to/App.js'. (in fact, take a look at the index.js file and you’ll see it being done!)

### Node Modules

What we've done above when we ran the `npm install` command was that we installed a module. This means that somebody has pushed a module (just like our `App` module above!) to `npm` (Node Package Manager), which we can then install and use in our code!

This way we can use React and build our app without having to globally attach anything, a big benefit in terms of understanding what is going on!

Alright, back to our weather app.

We'll need a bit of styling to make sure our app looks good. I've prepared that for you so you can focus on React, simply replace all the CSS in `App.css` with what is on <a target="_blank" href="styles.css">this page</a>.

We'll also need to be able to tell our app for which location we want the weather, so let's add a form with an input field and label that says "City, Country"!

{% filename %}App.js{% endfilename %}
```JS
class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Weather</h1>
        <form>
          <label>I want to know the weather for
            <input placeholder={"City, Country"} type="text" />
          </label>
        </form>
      </div>
    );
  }
}
```

> We nest the `input` inside the `label` so the input is focussed when users click on the label!

When entering something into the input field and pressing "Enter", the page refreshes and nothing happens. What we really want to do is fetch the data when a city and a country are input. Let's add an `onSubmit` handler to the `form` and a `fetchData` function to our component!

{% filename %}App.js{% endfilename %}
```JS
class App extends React.Component {
  fetchData = (evt) => {
    evt.preventDefault();
    console.log('fetch data!');
  };

  render() {
    return (
      <div>
        <h1>Weather</h1>
        <form onSubmit={this.fetchData}>
          <label>I want to know the weather for
            <input placeholder={"City, Country"} type="text" />
          </label>
        </form>
      </div>
    );
  }
}
```

By running `evt.preventDefault()` in fetchData (which is called when we press enter in the form), we tell the browser to not refresh the page and instead ignore whatever it wanted to and do what we tell it to. Right now, it logs "fetch data!" in the console over and over again whenever you submit the form. How do we get the entered city and country in that function though?

By storing the value of the text input in our local component state, we can grab it from that method. When we do that, we make our input a so-called "controlled input".

We'll store the currently entered location in `this.state.location`, and add a utility method to our component called `changeLocation` that is called `onChange` of the text input and sets the state to the current text:

{% filename %}App.js{% endfilename %}
```JS
class App extends React.Component {
  fetchData = (evt) => { /* … */ };

  changeLocation = (evt) => {
    this.setState({
      location: evt.target.value
    });
  };

  render() {
    return (
      <div>
        <h1>Weather</h1>
        <form onSubmit={this.fetchData}>
          <label>I want to know the weather for
            <input
              placeholder={"City, Country"}
              type="text"
              value={this.state.location}
              onChange={this.changeLocation}
            />
          </label>
        </form>
      </div>
    );
  }
}
```

As mentioned in [Meeting react](meeting-react.md), when saving anything to our local state, we have to predefine it. Let's do that:

{% filename %}App.js{% endfilename %}
```JS
class App extends React.Component {
  state = {
    location: ''
  };

  fetchData = (evt) => { /* … */ };

  changeLocation = (evt) => {
    this.setState({
      location: evt.target.value
    });
  };

  render() {
    return (
      <div>
        <h1>Weather</h1>
        <form onSubmit={this.fetchData}>
          <label>I want to know the weather for
            <input
              placeholder={"City, Country"}
              type="text"
              value={this.state.location}
              onChange={this.changeLocation}
            />
          </label>
        </form>
      </div>
    );
  }
}
```

In our fetchData function, we can then access `this.state.location` to get the current location:

{% filename %}App.js{% endfilename %}
```JS

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => {
    evt.preventDefault();
    console.log('fetch data for', this.state.location);
  };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

Now, whichever location you enter it should log "fetch data for MyCity, MyCountry"!
