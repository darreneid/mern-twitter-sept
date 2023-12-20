# Frontend Auth, Phases 3-4: Routes And Navigation

In Phases 3-4, you will configure your routes and set up your navigation bar.

## Phase 3: Routes and `MainPage`

It's time to configure your routes!

First, though, you need a component to show at a route. Make a `MainPage`
component that users will see when they land on the home page:

```js
// src/components/MainPage/MainPage.jsx

function MainPage() {
  return (
    <>
      <p>A Twitter Clone</p>
      <footer>
        Copyright &copy; 2023 MERN Twitter
      </footer>
    </>
  );
}

export default MainPage;
```

Now use `createBrowserRouter` and `RouterProvider` from `react-router-dom` to
create your `App` component in __src/App.jsx__. For your router, set up an
initial `AuthRoute` that displays `MainPage` at the root(`/`) path:

```js
// src/App.jsx

import { createBrowserRouter, RouterProvider, Outlet } from 'react-router-dom';
import { AuthRoute } from './components/Routes/Routes';
import MainPage from './components/MainPage/MainPage';

const router = createBrowserRouter([
  {
    path: "/",
    element: <AuthRoute component={MainPage} />
  }
]);

function App() {
  return <RouterProvider router={router} />;
}

export default App;
```

Now think about the remaining `AuthRoute`s and components
you'll need to complete your application. It may also be useful to have a
navigation bar that appears on every page. You will also need to render
components with forms that allow your users to sign up or log in.

Considering these needs, continue building out __App.jsx__, anticipating the
components that you will need to construct:

```js
// src/App.jsx

import { AuthRoute } from './components/Routes/Routes';
import NavBar from './components/NavBar/NavBar';

import MainPage from './components/MainPage/MainPage';
import LoginForm from './components/SessionForms/LoginForm';
import SignupForm from './components/SessionForms/SignupForm';

const Layout = () => {
  return (
    <>
      <NavBar />
      <Outlet />
    </>
  );
};

const router = createBrowserRouter([
  {
    element: <Layout />,
    children: [
      {
        path: "/",
        element: <AuthRoute component={MainPage} />
      },
      {
        path: "login",
        element: <AuthRoute component={LoginForm} />
      },
      {
        path: "signup",
        element: <AuthRoute component={SignupForm} />
      }
    ]
  }
]);

function App() {
  return <RouterProvider router={router} />;
}

export default App;
```

## Phase 4: Navigation bar

Your navigation bar appears on every page. It should contain either navigation
links or session links, depending on whether or not the user is logged in. Use
`session.user` in your state to determine which set of links to render.
(**Tip:** Use `useSelector`!) You also want to map your logout action to this
component so that your user always has the option to end their session.

Taking all of these requirements into consideration, your new `NavBar` component
should look something like this:

```js
// src/components/NavBar/NavBar.jsx

import { Link } from 'react-router-dom';
import { useSelector, useDispatch } from 'react-redux';
import './NavBar.css';
import { logout } from '../../store/session';

function NavBar () {
  const loggedIn = useSelector(state => !!state.session.user);
  const dispatch = useDispatch();
  
  const logoutUser = e => {
      e.preventDefault();
      dispatch(logout());
  };

  const getLinks = () => {
    if (loggedIn) {
      return (
        <div className="links-nav">
          <Link to={'/tweets'}>All Tweets</Link>
          <Link to={'/profile'}>Profile</Link>
          <Link to={'/tweets/new'}>Write a Tweet</Link>
          <button onClick={logoutUser}>Logout</button>
        </div>
      );
    } else {
      return (
        <div className="links-auth">
          <Link to={'/signup'}>Signup</Link>
          <Link to={'/login'}>Login</Link>
        </div>
      );
    }
  };

  return (
    <>
      <h1>MERN Twitter</h1>
      { getLinks() }
    </>
  );
}

export default NavBar;
```

You're well on your way! **Commit your code,** then head on to Phase 5 to
implement login and signup.
