**1) Describe the process, in a react/redux project, to fetch a set of data from a remote API endpoint, then to store that data into redux and then to present it on the screen.**

In order to achieve this we need:

- A React component that will render the fetched data on the screen.
- A React container that wraps the component with the Redux Provider.
- A Redux store that holds the application's state including the data that will be fetched from the remote API endpoint.
- A Redux reducer that updates the state of the store with the fetched data or fetching error, and supports 3 actions:
  - One dispatched when data fetching is initialized
  - One dispatched when data fetching is successfull and which will contain the fetched data as its payload
  - One dispatched when data has failed to fetch and which will include an error message as its payload

The process would go like this:

1. In the React container, fetch the server request to fetch the data, and immediately dispatch the first action.
2. The data will be fetched asynchronously, so while the promise resolves, use the first action to update the state in a way that triggers a loading indicator can be used to let the user know that data is being fetched.
3. When the fetching promise is resolved, if data is fetched successfully, dispatch the SUCCESS action with the fetched data as its payload. The reducer will update the state of the store with the new data. If, however, data fails to fetch, trigger the FAILURE action to instead update the state to include an error message that the React component can use to display on screen.
4. Assuming the data was fetched suggessfully, the Redux store would now contain the fetch data. Render the React component and pass it the data as props via the React container.

Below is some example code *only* intended for example purposes and not meant as a complete file:

```javascript
const initialState = { data: null, loading: false, error: null };

const reducer = (state, action) => {
  switch (action.type) {
    case 'FETCH_DATA_INITIALIZE':
      return { ...state, loading: true, error: null };
    case 'FETCH_DATA_SUCCESS':
      return { data: action.payload, loading: false, error: null };
    case 'FETCH_DATA_FAILURE':
      return { ...state, loading: false, error: action.payload };
    default:
      return state;
  }
};

const DataComponent = () => {
  const [state, dispatch] = useReducer(reducer, initialState);

  useEffect(() => {
    (async () => {
      try {
        dispatch({ type: 'FETCH_DATA_INITIALIZE' });
        const response = await fetch('https://examplesite.com/data');
        const data = await response.json();
        dispatch({ type: 'FETCH_DATA_SUCCESS', payload: data });
      } catch (error) {
        dispatch({ type: 'FETCH_DATA_FAILURE', payload: error.message });
      }
    })();
  }, []);

  return (
    <div>
      {state.loading && <p>Fetching data...</p>}
      {state.error && <p>{state.error}</p>}
      {state.data && (
        <ul>
          {state.data.map((item) => (
            <li key={item.id}>{item.name}</li>
          ))}
        </ul>
      )}
    </div>
  );
};
```

Alternatively, the actions can be created through [action creators](https://read.reduxbook.com/markdown/part1/04-action-creators.html) for higher code abstraction.

Meanwhile the container code could look something like this:

```javascript
const store = createStore(rootReducer);

const Root = () => (
  <Provider store={store}>
    <App />
  </Provider>
);

render(<Root />, document.getElementById('root'));
```

**2) Create a function `generateUrl` to generate a URL from given parameters:**

```
{
width: 360,
height: 300,
locale: 'en',
toolbar_bg: '',
interval: '3h',
pair: 'BTC_USD',
}
```

You can use any lib but the generated result should be  "http://testurl.bitfinx.com/?height=300&interval=3h&locale=en&pair=BTC_USD&width=360"

More parameters are planned to be added/removed later and the the result should neglect the empty params (ex: should not include toolbar_bg in URL when its value is empty).

**SOLUTION**

```javascript
function generateUrl (baseUrl, urlParams) {
  const params = [];
  const isValueAllowed = (value) => 
    ['string', 'number', 'boolean'].includes(typeof value) && value !== '' && !isNaN(value);
  for (const [key, value] of Object.entries(urlParams)) {
    if (isValueAllowed(value)) {
      params.push(`${encodeURIComponent(key)}=${encodeURIComponent(value)}`);
    }
  }
  return `${baseUrl}${params.length ? '?' : ''}${params.join('&')}`;
}

```

- The suggested solution additionally curates the values to allow only certain types, as well as prevents `NaN`.

**3) Apply some refactoring to improve the code of the following function. Explain the reasons behind your changes and which benefit they bring into the code.**

```
var volumeSetup = function () {
// setup volume unit interface
var volumeUnit = window.APP.util.getSettings('ticker_vol_unit').toUpperCase();
var element = null;
if (volumeUnit === 'FIRSTCCY') {
element = $('#tickervolccy_0');
} else if (volumeUnit === 'USD') {
element = $('#tickervolccy_USD');
} else if (volumeUnit === 'BTC') {
element = $('#tickervolccy_BTC');
} else if (volumeUnit === 'ETH') {
element = $('#tickervolccy_ETH');
}
if (element) {
element.prop("checked", true);
}
// override currencies list
var result = window.APP.util.initCurrenciesList()
return result
}
```

**SOLUTION**

```javascript
function volumeSetup(util = window.APP.util, selectorMap) {
  // setup volume unit interface
  const volumeUnit = util.getSettings('ticker_vol_unit').toUpperCase();
  const unitSelectorMap = selectorMap || {
    'FIRSTCCY': '#tickervolccy_0',
    'USD': '#tickervolccy_USD',
    'BTC': '#tickervolccy_BTC',
    'ETH': '#tickervolccy_ETH'
  }
  const element = $(unitSelectorMap[volumeUnit]);
  element?.prop("checked", true);
  // override currencies list
  const result = util.initCurrenciesList();
  return result;
}
```

- The proposed refactoring reduces the lines of code, avoids code repetition, and improves the overall scalability and abstraction of the function.
- The new function accepts 2 params with default values that can be overwritten, which makes the function higher purity and testeable.
