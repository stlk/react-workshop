## Creating the app

### Prerequisites

Node.js is a JavaScript runtime for your terminal. If you don’t have it installed already (check by running node -v in your terminal, which should print a version number) head over to [nodejs.org](http://nodejs.org) and install it.

### Project setup

We can use a tool called `create-react-app` to do the hard work of configuring the app. It includes all the necessary build tools and transpilation steps to just get stuff done.

Let’s install it with npm:
```sh
npm install -g create-react-app
```

As soon as that’s finished you now have access to the `create-react-app` command in your terminal! Let’s create our weather app:

```sh
create-react-app weather-app
```

The argument to `create-react-app`, in our case `weather-app`, tells the utility what to name the folder it’ll create. Since we’re creating a weather app, weather-app seems like a solid choice!

Take a look into the `src/index.js` file and you’ll see something like this:

{% filename %}index.js{% endfilename %}
```js
import React from 'react';
import ReactDOM from 'react-dom';
import App from './App';
import './index.css';

ReactDOM.render(
  <App />,
  document.getElementById('root')
);
```

Thankfully, `create-react-app` includes a simple server so instead of having to open the `index.html` file manually we can simply run `npm run start` in the `weather-app` directory and see our application at `localhost:3000`!

If you take a look into the `src/App.js` component, you’ll see a bunch of boilerplate code in there. Delete the `import logo from './logo.svg';` (and the logo.svg file if you want) and all of the JSX, and instead render a heading saying “Weather”:

{% filename %}App.js{% endfilename %}
```js
import React from 'react';
import './App.css';

class App extends React.Component {
  render() {
    return (
      <h1>Weather</h1>
    );
  }
}

export default App;
```

Save the file, go back to your browser and you should see a heading saying “Weather”! Now let's look into how imports and exports work.
