# Manageable CSS

- components and their styles

Our CSS is stored in `App.css`. Let's try to create `Wrapper` and `Title` components which encapsulate style specification.

```css
h1 {
  text-align: center;
}

.wrapper {
  max-width: 680px;
  margin: 0 auto;
}
```

## Styled components

First, we need to install `styled-components`:

```sh
npm install --save styled-components
```

Then in our `App.js` lets import it and define two components.

{% filename %}App.js{% endfilename %}
```js
import styled from 'styled-components';


const Title = styled.h1`
  text-align: center;
`;

const Wrapper = styled.div`
  max-width: 680px;
  margin: 0 auto;
`;
```

And then use them instead of `<h1>` and `<div className="wrapper"`:

{% filename %}App.js{% endfilename %}
```jsx
export class App extends React.Component {

  fetchData = (evt) => {
    /* … */
  }

  changeLocation = (evt) => {
      /* … */
  }

  onPlotClick = (data) => {
    /* … */
  }

  render() {
      /* … */
      return (
        <div>
          <Title>Weather</Title>
          <form onSubmit={this.fetchData}>
            <label>I want to know the weather for
              <input
                placeholder={"City, Country"}
                type="text"
                value={this.props.redux.get('location')}
                onChange={this.changeLocation}
              />
            </label>
          </form>
          {/*
            Render the current temperature and the forecast if we have data
            otherwise return null
          */}
          {this.props.redux.getIn(['data', 'list']) ?
            <Wrapper>
              {/* Render the current temperature if no specific date is selected */}
              <p className="temp-wrapper">
                <span className="temp">
                  {this.props.redux.getIn(['selected', 'temp']) ? this.props.redux.getIn(['selected', 'temp']) : currentTemp}
                </span>
                <span className="temp-symbol">°C</span>
                <span className="temp-date">
                  {this.props.redux.getIn(['selected', 'temp']) ? this.props.redux.getIn(['selected', 'date']) : ''}
                </span>
              </p>
              <h2>Forecast</h2>
              <Plot
                xData={this.props.redux.get('dates')}
                yData={this.props.redux.get('temps')}
                onPlotClick={this.onPlotClick}
                type="scatter"
              />
            </Wrapper>
           : null}
        </div>
      );
  }
}
```
