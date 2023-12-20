# React Setup

It's time to set up the frontend of your application!

- In the **root directory** of your project, run
  
  ```sh
  npx tiged appacademy/aa-react18-vite-template frontend
  ```

  to install a new React application in a new folder called __frontend__.

  Your folder structure should now look like this:

  ```plaintext
    <root>
      ├── backend
      └── frontend
  ```

  Note that **this structure is different from the structure that you used for
  your Full Stack Project**, where the __frontend__ folder was inside the
  __backend__ folder. In this project, __frontend__ and __backend__ are distinct
  directories at the same level.

- `cd` into your __frontend__ folder and `npm install` your standard Redux and
  Router packages:
  - `react-redux` - React components and hooks for Redux
  - `react-router-dom` - Routing for React
  - `redux` - Redux
  - `redux-thunk` - Redux thunk

- Also install `redux-logger` as a `devDependency` (so it won't be included in
  production):

  ```sh
  npm install -D redux-logger
  ```

- When setting up routes for your React app, you want to be able to write
  something like `/api/users/:id` rather than having to type the full path. To
  enable this, add a `proxy` key-value pair to the `server` key in __vite.config.js__:
  
  ```js
  server: {
    proxy: {
      "/api": "http://localhost:5000"
    }
  }
  ```

- Finally, open __frontend/index.html__ and change the `title` from `App Academy
  Template` to `MERN Twitter`. Similarly, change the `name` in
  __frontend/package.json__ to `"mern-twitter-frontend"`.

## File Structure

Since you have already covered React earlier in the App Academy curriculum, this
tutorial will not do a deep dive into React or Redux. It will, however, walk
through the steps of setting up the Redux cycle so that you can authenticate
users, access tweets, and view a single user's tweets.

Start by adding __components__ and __store__ folders to the __frontend/src__
directory.

## Store

Let's configure the Redux store. This step should be familiar to you after your
experience with the Full Stack Project earlier in the curriculum.

Within the __store__ directory, create a new file called __store.js__:

```js
// store/store.js

import { createStore, applyMiddleware, compose, combineReducers } from 'redux';
import thunk from 'redux-thunk';

const rootReducer = combineReducers({
  // Add reducers here
});

let enhancer;
if (import.meta.env.MODE === "production") {
  enhancer = applyMiddleware(thunk);
} else {
  const logger = (await import("redux-logger")).default;
  const composeEnhancers =
    window.__REDUX_DEVTOOLS_EXTENSION_COMPOSE__ || compose;
  enhancer = composeEnhancers(applyMiddleware(thunk, logger));
}

const configureStore = (preloadedState) => {
  return createStore(rootReducer, preloadedState, enhancer);
};

export default configureStore;
```

As you likely recall, this code sets up the Redux DevTools and `logger`
middleware to run in non-`production` environments. It also configures your
store with the possibility of passing in a preloaded state.

## Entry file

Conclude this setup by configuring your entry file, __src/main.jsx__. Configure your store and wrap `App` in the Redux
`Provider`. It should look something like this:

```js
// src/main.jsx

import React from 'react';
import ReactDOM from 'react-dom/client';
import { Provider } from 'react-redux';
import App from './App';
import './index.css';
import configureStore from './store';

const store = configureStore();

ReactDOM.createRoot(document.getElementById('root')).render(
  <React.StrictMode>
    <Provider store={store}>
      <App />
    </Provider>
  </React.StrictMode>
);
```

Your frontend is all set up! **Commit your code** and head on to Frontend Auth,
Phase 1.
