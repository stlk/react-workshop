## Fetching data

Let's get into fetching data. Instead of console logging a text, we need to get some weather information. We'll be using the <a target="_blank" href="http://openweathermap.org/api">OpenWeatherMap API</a> for this task, which is a free service that provides access to data for basically all locations all around the world. You'll need to get an API key from it, so head over to <a target="_blank" href="http://openweathermap.org/api">openweathermap.org/api</a>, press "Sign Up" in the top bar and register for a free account:

As soon as you've done that go to your API key page by going to <a target="_blank" href="http://home.openweathermap.org/api_keys">home.openweathermap.org/api_keys</a>, copy the `API key` from there and keep it somewhere safe.

Now that we have access to all the weather data our heart could desire, let's get on with our app!

Inside our `fetchData` function, we'll have to make a request to the API. I like to use a npm module called `xhr` for this, a wrapper around the JavaScript XMLHttpRequest that makes said requests a lot easier. Run:

```Sh
npm install --save xhr
```

to get it! While that's installing, `import` it in your App component at the top:

{% filename %}App.js{% endfilename %}
```JS
import React from 'react';
import './App.css';
import xhr from 'xhr';

class App extends React.Component { /* … */ };
```

To get the data, the structure of the URL we'll request looks like this:

```
http://api.openweathermap.org/data/2.5/forecast?q=CITY,COUNTRY&APPID=YOURAPIKEY&units=metric
```

Replace `CITY,COUNTRY` with a city and country combination of choice, replace `YOURAPIKEY` with your copied API key and open that URL in your browser. (it should look something like this: `http://api.openweathermap.org/data/2.5/forecast?q=Vienna,Austria&APPID=asdf123&units=metric`)

What you'll get is a JSON object that has the following structure:

```JS
"city": {
  "id": 2761369,
  "name": "Vienna",
  "coord": {
    "lon": 16.37208,
    "lat": 48.208488
  },
  "country": "AT",
  "population": 0,
  "sys": {
    "population": 0
  }
},
"cod": "200",
"message": 0.0046,
"cnt": 40,
"list": [ /* Hundreds of objects here */ ]
```

The top level `list` array contains time sorted weather data reaching forward 5 days. One of those weather objects looks like this: (only relevant lines shown)

```JS
{
  "dt": 1460235600,
  "main": {
    "temp": 6.94,
    "temp_min": 6.4,
    "temp_max": 6.94
  },
  "weather": [
    {
      "main": "Rain",
      /* …more data here */
    }
  ],
  /* …more data here */
}
```

The five properties we care about are: `dt_txt`, the time of the weather prediction, `temp`, the expected temperature, `temp_min` and `temp_max`, the, respectively, minimum and maximum expected temperature, and `weather[0].main`, a string description of the weather at that time. OpenWeatherMap gives us a lot more data than that though, and I encourage you to snoop around a bit more and see what you could use to make the application more comprehensive!

Now that we know what we need, let's get down to it – how do we actually fetch the data here? (by now `xhr` should have finished installing)

The general usage of `xhr` looks like this:

```JS
xhr({
  url: 'someURL'
}, function (err, data) {
  /* Called when the request is finished */
});
```

As you can see, everything we really need to take care of is constructing the url and saving the returned data somewhere!

We know that the URL has a prefix that's always the same, `http://api.openweathermap.org/data/2.5/forecast?q=`, and a suffix that's always the same, `&APPID=YOURAPIKEY&units=metric`. The sole thing we need to do is insert the location the user entered into the URL!

> You can also change the units you get back by setting `units` to `imperial`: `http://api.openweathermap.org/data/2.5/forecast?q=something&APPID=YOURAPIKEY&units=imperial`

Now, if you're thinking this through you know what might happen – the user might enter spaces in the input! URLs with spaces aren't valid, so it wouldn't work and everything would break! While that is true, JavaScript gives us a very handy method to escape non-URL-friendly characters. It is called `encodeURIComponent()`, and this is how one uses it:

```JS
encodeURIComponent('My string with spaces'); // -> 'My%20string%20with%20spaces'
```

Combine this method with the URL structure we need, the `xhr` explanation and the state of the component and we've got all the ingredients we need to get the data from the server!

> Please note that you might need to restart the development server (`CTRL-C` to stop it, `npm start` to start it again) for it to know that a new module was installed!

First, let's encode the location from the state:

{% filename %}App.js{% endfilename %}
```JS
import React from 'react';
import './App.css';
import xhr from 'xhr';

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => {
    evt.preventDefault();

    var location = encodeURIComponent(this.state.location);
  };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

Second, let's construct the URL we need using that escaped location:

{% filename %}App.js{% endfilename %}
```JS
import React from 'react';
import './App.css';
import xhr from 'xhr';

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => {
    evt.preventDefault();

    var location = encodeURIComponent(this.state.location);
    var token = 'YOURAPIKEY'
    var url = `http://api.openweathermap.org/data/2.5/forecast?q=${location}&APPID=${token}&units=metric`
  };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

The last thing we need to do to get the data from the server is call `xhr` with that url!

{% filename %}App.js{% endfilename %}
```JS
import React from 'react';
import './App.css';
import xhr from 'xhr';

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => {
    evt.preventDefault();

    var location = encodeURIComponent(this.state.location);
    var token = 'YOURAPIKEY'
    var url = `http://api.openweathermap.org/data/2.5/forecast?q=${location}&APPID=${token}&units=metric`

    xhr({
      url: url
    }, function (err, data) {
      /* …save the data here */
    });
  };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

Since we want React to rerender our application when we've loaded the data, we'll need to save it to the state of our `App` component. Also, network request data is a string, so we'll need to parse that with `JSON.parse` to make it an object.

{% filename %}App.js{% endfilename %}
```JS
import React from 'react';
import './App.css';
import xhr from 'xhr';

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => {
    evt.preventDefault();

    var location = encodeURIComponent(this.state.location);
    var token = 'YOURAPIKEY'
    var url = `http://api.openweathermap.org/data/2.5/forecast?q=${location}&APPID=${token}&units=metric`

    xhr({
      url: url
    }, (err, data) => {
      this.setState({
        data: JSON.parse(data.body)
      });
    });
  };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

> Note: Using `(err, data) => {` instead of `function (err, data) {` is necessary because otherwise `this.setState` wouldn't work.

Let's define that `data` in our state:

{% filename %}App.js{% endfilename %}
```JS
import React from 'react';
import './App.css';
import xhr from 'xhr';

class App extends React.Component {
  state = {
    location: '',
    data: {}
  };

  fetchData = (evt) => { /* … */ };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

Now that we've got the weather data for the location we want in our component state, we can use it in our render method! Remember, the data for the current weather is in the `list` array, sorted by time. The first element of said array is thus the current temperature, so let's try to render that first:

{% filename %}App.js{% endfilename %}
```JS
import React from 'react';
import './App.css';
import xhr from 'xhr';

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => { /* … */ };
  changeLocation = (evt) => { /* … */ };

  render() {
    var currentTemp = 'not loaded yet';
    if (this.state.data.list) {
      currentTemp = this.state.data.list[0].main.temp;
    }
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
        <p className="temp-wrapper">
          <span className="temp">{ currentTemp }</span>
          <span className="temp-symbol">°C</span>
        </p>
      </div>
    );
  }
}
```

Go open that in your browser, enter your current location and you'll see the current temperature! 🎉 Awesome!

## Summary of this chapter

We learned how to structure our application and then we created a controlled text input and used that to fetch our first data using `xhr`!

Now that we have the current temperature, we need to render the forecast!

## Additional Material

- <a target="_blank" href="https://github.com/facebookincubator/create-react-app/blob/623e1bd189eeb154d52dad8d74dee251e54c8211/template/README.md">Official create-react-app docs</a>
- <a target="_blank" href="https://github.com/Raynos/xhr">Official xhr docs</a>
- <a target="_blank" href="http://readwrite.com/2013/09/19/api-defined/">What APIs Are And Why They’re Important</a>
