## Plotly

To use `plotly.js`, we need to add it to our application first. Copy and paste this snippet into our `index.html`:

```HTML
<script src="https://cdn.plot.ly/plotly-1.8.0.min.js"></script>
```

Including this script gives us access to the `Plotly` variable in our code. Using `Plotly.newPlot`, we can easily create graphs to showcase the weather data.

The second argument is an array of objects with a few properties for our plot, with `x`, `y` and `type` being the most relevant for us. Plotly.js makes it easy to create a wide variety of plots, the one we care about the most at the moment is a `scatter` plot. See the documentation <a href="https://plot.ly/javascript/line-and-scatter/" target="_blank">here</a> for some examples. As you can see, it's perfect for a weather forecast!

This is what our `Plotly.newPlot` call will look like:

```jsx
Plotly.newPlot('someDOMElementId', [{
  x: ourXAxisData,
  y: ourYAxisData,
  type: 'scatter'
}], {
  margin: {
    t: 0, r: 0, l: 30
  },
  xaxis: {
    gridcolor: 'transparent'
  }
}, {
  displayModeBar: false
});
```

As you can see, we also pass in some styling information as the third argument (we specify a few margins and hide the xaxis grid lines), and some options as the fourth argument. (we hide the mode bar)

Plotly.js has tons of options, I encourage you to check out the <a target="_blank" href="https://plot.ly/javascript/">excellent documentation</a> and play around with a few of them!

To actually get this done though, we need to create a new component first. We'll call it `Plot` (what a surprise!), so add a new file in your `src/` folder called `Plot.js`, and render just a div:

{% filename %}Plot.js{% endfilename %}
```jsx
import React from 'react';

class Plot extends React.Component {
  render() {
    return (
      <div id="plot"></div>
    );
  }
}

export default Plot;
```

> As you can see, I've added a `div` with an ID of `plot` above. This is the DOM element we'll reference in our `Plotly.newPlot` call!

Now, the problem we have here is that if we called `Plotly.newPlot` in our `render` method, it would be called over and over again, possibly multiple times per second! That's not optimal, we really want to call it once when we get the data and leave it be afterwards – how can we do that?

Thankfully, React gives us a lifecycle method called `componentDidMount`. It is called once when the component was first rendered, and never afterwards; perfect for our needs! Let's create a `componentDidMount` method and call `Plotly.newPlot` in there and pass it the ID of our `div`, `plot`, as the first argument:

{% filename %}Plot.js{% endfilename %}
```jsx
import React from 'react';

class Plot extends React.Component {
  componentDidMount() {
    Plotly.newPlot('plot');
  }

  render() {
    return (
      <div id="plot"></div>
    );
  }
}

export default Plot;
```

You'll now see a warning in the console though – "Plotly is not defined". Since we injected `Plotly` globally via the script tag we need to tell `create-react-app` that this variable exists by adding a comment at the top of the file saying `/* global Plotly */` like so:

{% filename %}Plot.js{% endfilename %}
```jsx
/* global Plotly */
import React from 'react';

class Plot extends React.Component {
  componentDidMount() {
    Plotly.newPlot('plot');
  }

  render() {
    return (
      <div id="plot"></div>
    );
  }
}

export default Plot;
```

That alone won't do much though, we need to give it data too! The problem is that we need the data for the x-axis and the y-axis to be separate, but the data we get from the OpenWeatherMap API doesn't make that distinction. This means we need to shape our data a little bit to suit our needs. What we want is human readable dates on the x-axis, and the degrees at that time on the y-axis!

Let's jump back to our `App` component, and start changing the data a little bit. We'll do that in the `fetchData` method, so we only recalculate the data when new one comes in and not on every render. (which would possibly mean shaping the data every second or more!) This is what happens when the data comes back in at the moment:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => {
    /* … */

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

Instead of just saving the raw data in our `xhr` callback, let's shape the data into a form we can use it in and save both the raw and the formed data in our component state.

What we really care about is `data.list[element].dt_txt`, a human-readable timestamp, and `data.list[element].main.temp`, the temperature at that time.

Let's loop through all the weather information we have, making two arrays of different data. We'll use the `push` method of arrays, which adds an element to the end of an array. Let's fill one with the timestamps, and another array with the temperatures:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => {
    /* … */

    xhr({
      url: url
    }, (err, data) => {
      var body = JSON.parse(data.body);
      var list = body.list;
      var dates = [];
      var temps = [];
      for (var i = 0; i < list.length; i++) {
        dates.push(list[i].dt_txt);
        temps.push(list[i].main.temp);
      }

      this.setState({
        data: body
      });
    });
  };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

Now we have exactly what we want, we just need to save it to our component state! Let's call the two properties of our state `dates` and `temperatures`:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = { /* … */ };

  fetchData = (evt) => {
    /* … */

    xhr({
      url: url
    }, (err, data) => {
      var body = JSON.parse(data.body);
      var list = body.list;
      var dates = [];
      var temps = [];
      for (var i = 0; i < list.length; i++) {
        dates.push(list[i].dt_txt);
        temps.push(list[i].main.temp);
      }

      this.setState({
        data: body,
        dates: dates,
        temps: temps
      });
    });
  };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

We also need to add those new properties to our initial state:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = {
    location: '',
    data: {},
    dates: [],
    temps: []
  };

  fetchData = (evt) => { /* … */ };

  changeLocation = (evt) => { /* … */ };

  render() { /* … */ }
}
```

Now that we have that data saved in our component state, we can render our plot! `import` our `Plot` component, and pass it `this.state.dates` as the x-axis data, `this.state.temps` as the y-axis data and we'll also pass it a `type` prop of `"scatter"`!

{% filename %}App.js{% endfilename %}
```jsx
import React from 'react';
import './App.css';
import xhr from 'xhr';

import Plot from './Plot.js';

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
        <div className="wrapper">
          <p className="temp-wrapper">
            <span className="temp">{ currentTemp }</span>
            <span className="temp-symbol">°C</span>
          </p>
          <h2>Forecast</h2>
          <Plot
            xData={this.state.dates}
            yData={this.state.temps}
            type="scatter"
          />
        </div>
      </div>
    );
  }
}
```

We only want to render the current temperature and the forecast when we have data though, so let's add a ternary operator to check that `this.state.data.list` exists:

{% filename %}App.js{% endfilename %}
```jsx

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
        {/*
          Render the current temperature and the forecast if we have data
          otherwise return null
        */}
        {(this.state.data.list) ? (
          <div className="wrapper">
            <p className="temp-wrapper">
              <span className="temp">{ currentTemp }</span>
              <span className="temp-symbol">°C</span>
            </p>
            <h2>Forecast</h2>
            <Plot
              xData={this.state.dates}
              yData={this.state.temps}
              type="scatter"
            />
          </div>
        ) : null}

      </div>
    );
  }
}
```

If you try doing this now, you still won't see a plot, do you know why? Because we aren't using the data we passed to our `Plot` component! This is what it looks like at the moment:

{% filename %}Plot.js{% endfilename %}
```jsx
import React from 'react';

class Plot extends React.Component {
  componentDidMount() {
    Plotly.newPlot('plot');
  }

  render() {
    return (
      <div id="plot"></div>
    );
  }
}

export default Plot;
```

Let's make this work by adapting the `Plotly.newPlot` call. We need to pass our styling and options, and `this.props.xData`, `this.props.yData` and `this.props.type`:

{% filename %}Plot.js{% endfilename %}
```jsx
import React from 'react';

class Plot extends React.Component {
  componentDidMount() {
    Plotly.newPlot('plot', [{
      x: this.props.xData,
      y: this.props.yData,
      type: this.props.type
    }], {
      margin: {
        t: 0, r: 0, l: 30
      },
      xaxis: {
        gridcolor: 'transparent'
      }
    }, {
      displayModeBar: false
    });
  }

  render() {
    return (
      <div id="plot"></div>
    );
  }
}

export default Plot;
```

Awesome! Normally, creating a graph like this manually would take ages, but Plotly.js makes it incredibly easy!

There is one problem though: When we change the city and refetch data, the graph doesn't update. This is the case because we're solely using the `componentDidMount` lifecycle method, which is only ever called once when the component mounts. We also need to draw the plot again when new data comes in, i.e. when the component did update! (*hinthint*)

As you might have guessed, we can use the `componentDidUpdate` lifecycle method of our `Plot` component to fix this:

{% filename %}Plot.js{% endfilename %}
```jsx
import React from 'react';

class Plot extends React.Component {
  componentDidMount() {
    Plotly.newPlot('plot', [{
      x: this.props.xData,
      y: this.props.yData,
      type: this.props.type
    }], {
      margin: {
        t: 0, r: 0, l: 30
      },
      xaxis: {
        gridcolor: 'transparent'
      }
    }, {
      displayModeBar: false
    });
  }

  componentDidUpdate() {
    Plotly.newPlot('plot', [{
      x: this.props.xData,
      y: this.props.yData,
      type: this.props.type
    }], {
      margin: {
        t: 0, r: 0, l: 30
      },
      xaxis: {
        gridcolor: 'transparent'
      }
    }, {
      displayModeBar: false
    });
  }

  render() {
    return (
      <div id="plot"></div>
    );
  }
}

export default Plot;
```

Trying this out, it works perfectly! There is one tiny improvement, code wise, that could be done. Instead of copy and pasting the `Plotly.newPlot` call (which is identical), we should factor that out into a `drawPlot` method and call `this.drawPlot` from `componentDidMount/Update`:

{% filename %}Plot.js{% endfilename %}
```jsx
import React from 'react';

class Plot extends React.Component {
  drawPlot = () => {
    Plotly.newPlot('plot', [{
      x: this.props.xData,
      y: this.props.yData,
      type: this.props.type
    }], {
      margin: {
        t: 0, r: 0, l: 30
      },
      xaxis: {
        gridcolor: 'transparent'
      }
    }, {
      displayModeBar: false
    });
  }

  componentDidMount() {
    this.drawPlot();
  }

  componentDidUpdate() {
    this.drawPlot();
  }

  render() {
    return (
      <div id="plot"></div>
    );
  }
}

export default Plot;
```

Beautiful, and works perfectly too!

Let's add one more feature to our weather application. When clicking on a specific point of our graph, we want to show the user in text the temperature at that date!

The first thing we need to do is add an event listener to our graph. Thankfully, Plotly gives us a handy `plotly_click` event to listen to, like so:

```jsx
// Called when a plot inside the DOM element with the id "someID" is clicked
document.getElementById('someID').on('plotly_click', function(data) {
  /* …do something here with the data… */
});
```

The nice thing about `plotly_click` is that it doesn't pass you the event, it passes you a very useful `data` object. We care about two particular properties of that `data` object:

```jsx
{
  "points": [{
    "x": "2016-07-29 03",
    "y": 17.4,
    /* …more data here… */
  }]
}
```

These tell us which date was clicked on and what the relevant temperature was, exactly what we want! We'll pass a function down to the `Plot` component called `onPlotClick` that will get called when the `plotly_click` event is fired, i.e. when a point on our forecast is clicked on.

Let's start off by binding that event listener in our `Plot` component. In our `drawPlot` method bind the `plotly_click` event to `this.props.onPlotClick`!

{% filename %}Plot.js{% endfilename %}
```jsx

class Plot extends React.Component {
  drawPlot = () => {
    Plotly.newPlot( /* … */ );
    document.getElementById('plot').on('plotly_click', this.props.onPlotClick);
  };

  componentDidMount() { /* … */ }
  componentDidUpdate() { /* … */ }
  render() { /* … */ }
}

export default Plot;
```

Perfect, but running this will not work since we don't pass an `onPropClick` prop to `Plot`. Let's jump to our `App` component and change that. First, we pass an `onPlotClick` prop to our `Plot` component calling our `App` component's (currently missing) `this.onPropClick` method:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = { /* … */ };
  fetchData = (evt) => { /* … */ };
  changeLocation = (evt) => { /* … */ };

  render() {
    /* … */
    return (
      { /* … */ }
      <Plot
        xData={this.state.dates}
        yData={this.state.temps}
        onPlotClick={this.onPlotClick}
        type="scatter"
      />
      { /* … */ }
    );
  }
}
```

Then we add a first version of the `onPlotClick` method to our `App` component where we only log out the passed `data`:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = { /* … */ };
  fetchData = (evt) => { /* … */ };
  changeLocation = (evt) => { /* … */ };
  onPlotClick = (data) => {
    console.log(data);
  };

  render() {
    /* … */
    return (
      { /* … */ }
      <Plot
        xData={this.state.dates}
        yData={this.state.temps}
        onPlotClick={this.onPlotClick}
        type="scatter"
      />
      { /* … */ }
    );
  }
}
```

Now try opening your application, select a city and, when the forecast has rendered, click on a specific data point in the plot. If you see an object logged in your console containing an array called `points`, you're golden!

Instead of logging the data, we now want to save that data in our state. Let's add a new object to our initial state called `selected`, which contains a `date` and a `temp` field. The date field will be an empty string by default, and the temp `null`:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = {
    location: '',
    data: {},
    dates: [],
    temps: [],
    selected: {
      date: '',
      temp: null
    }
  };
  fetchData = (evt) => { /* … */ };
  changeLocation = (evt) => { /* … */ };
  onPlotClick = (data) => {
    console.log(data);
  };

  render() {
    /* … */
    return (
      { /* … */ }
      <Plot
        xData={this.state.dates}
        yData={this.state.temps}
        onPlotClick={this.onPlotClick}
        type="scatter"
      />
      { /* … */ }
    );
  }
}
```

Now, when our `onPlotClick` method is called we'll set the `selected.date` to `data.points[0].x`, and the the `selected.temp` to `data.points[0].x`:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = {
    location: '',
    data: {},
    dates: [],
    temps: [],
    selected: {
      date: '',
      temp: null
    }
  };
  fetchData = (evt) => { /* … */ };
  changeLocation = (evt) => { /* … */ };
  onPlotClick = (data) => {
    if (data.points) {
      this.setState({
        selected: {
          date: data.points[0].x,
          temp: data.points[0].y
        }
      });
    }
  };

  render() {
    /* … */
    return (
      { /* … */ }
      <Plot
        xData={this.state.dates}
        yData={this.state.temps}
        onPlotClick={this.onPlotClick}
        type="scatter"
      />
      { /* … */ }
    );
  }
}
```

Now that we have the necessary data in our state, we need to do something with it! Let's render some text saying "The current temperature on some-date is some-temperature°C!" if we have a date selected, and otherwise show the current date. We thus need to adapt the `render` method of our `App` component to include that. We check if `this.state.selected.temp` exists (i.e. isn't `null`, the default value), and if it does we render the text with `this.state.selected`:

{% filename %}App.js{% endfilename %}
```jsx

class App extends React.Component {
  state = { /* … */ };
  fetchData = (evt) => { /* … */ };
  changeLocation = (evt) => { /* … */ };
  onPlotClick = (data) => { /* … */ };

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
        {(this.state.data.list) ? (
          <div className="wrapper">
            {/* Render the current temperature if no specific date is selected */}
            <p className="temp-wrapper">
              <span className="temp">
                { this.state.selected.temp ? this.state.selected.temp : currentTemp }
              </span>
              <span className="temp-symbol">°C</span>
              <span className="temp-date">
                { this.state.selected.temp ? this.state.selected.date : ''}
              </span>
            </p>
            <h2>Forecast</h2>
            <Plot
              xData={this.state.dates}
              yData={this.state.temps}
              onPlotClick={this.onPlotClick}
              type="scatter"
            />
          </div>
        ) : null}

      </div>
    );
  }
}
```

Try opening your app again and clicking on a point on the graph, and you'll see our new functionality! There is one small user experience improvement we could do. When switching to a new city, the text persists because `this.state.selected.temp` still references the old data—in reality want to show the current temperature though!

To fix this, we set `selected` back to the default values in our `fetchData` method when the request has returned data:

```jsx
import React from 'react';
import xhr from 'xhr';

import Plot from './Plot';

var App = React.createClass({
  getInitialState: function() { /* … */ },
  fetchData: function(evt) {
    /* … */
    xhr({
      url: url
    }, (err, data) => {

      /* … */
      /* Save the data, and reset the selected time to the default values */
      this.setState({
        data: body,
        dates: dates,
        temps: temps,
        selected: {
          date: '',
          temp: null
        }
      });
    });
  },
  onPlotClick: function(data) { /* … */ },
  changeLocation: function(evt) { /* … */ },
  render: function() { /* … */ }
});

export default App;
```

Perfect, this now works beautifully! As you can see, another huge benefit of Plotly.js is that it makes interactivity really easy in combination with React.

Congratulations, you've built your first working application! 🎉

## Summary of this chapter

We've created a new `Plot` component, shaped the data we get from the OpenWeatherMap API to suit our needs and used Plotly.js to render a beautiful and interactive 5 day weather forecast!

## Additional Material

- <a href="https://plot.ly/javascript/" target="_blank">Official plotly.js docs</a>
- <a href="http://openweathermap.org/api" target="_blank">OpenWeatherMap API</a>
- <a href="http://www.jsgraphs.com/" target="_blank">JavaScript Graphing Library Comparison</a>
