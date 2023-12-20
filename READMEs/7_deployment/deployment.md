# Deploying Your Express + React App To Render.com

Before you begin deploying, **make sure to remove any `console.log`s or
`debugger`s in any production code**. Search your entire project folder to see
if you are using them anywhere.

You will set up [Render.com] to run a production, not development, version of
your application. In the following phases, you will accordingly configure your
application to work in production.

If you do not yet have a Render account, sign up for one [here][render-signup].

## Phase 1: Setting up your Express + React application

Right now, your React application is on a different localhost port (5173) than
your Express application (5000). However, since your React application consists
only of static files that don't need to bundled continuously with changes in
production, your Express application can serve the static React assets in
production too. These static files live in the __frontend/dist__ folder after
running `npm run build` in the __frontend__ folder. You want to tell Express to
serve up these frontend files for any non-`/api` route.

### Configure your Express backend

First, set your node version. Open the __package.json__ file in your __backend__
folder. Add an `"engines"` key that specifies the version of Node to use. (Run
`node -v` in your terminal to see which version you are using.) It is
recommended to specify only the major version, leaving Render free to install
the latest compatible minor version (which often contains security updates). It
should look something like this:

```json
  "engines": {
    "node": "18.x"
  },
```

> For alternative ways of specifying the Node version on Render, see
> [here][render-node].

To configure your Express backend to serve up your frontend files, you need to
adjust your __backend/app.js__ file. In brief, you want to serve the React
application's static __index.html__ file (along with the `CSRF-TOKEN` cookie!)
at the root route. Then serve up all the React application's static files using
the `express.static` middleware. Finally, serve the __index.html__ and set the
`CSRF-TOKEN` cookie again on all routes that don't start in `/api`. Adding this
functionality will look like this:

```js
// backend/app.js

// ...
// These lines should already be present.
app.use('/api/tweets', tweetsRouter);
app.use('/api/users', usersRouter);
app.use('/api/csrf', csrfRouter);

// Serve static React build files statically in production
if (isProduction) {
  const path = require('path');
  // Serve the frontend's index.html file at the root route
  app.get('/', (req, res) => {
    res.cookie('CSRF-TOKEN', req.csrfToken());
    res.sendFile(
      path.resolve(__dirname, '../frontend', 'dist', 'index.html')
    );
  });

  // Serve the static assets in the frontend's dist folder
  app.use(express.static(path.resolve("../frontend/dist")));

  // Serve the frontend's index.html file at all other routes NOT starting with /api
  app.get(/^(?!\/?api).*/, (req, res) => {
    res.cookie('CSRF-TOKEN', req.csrfToken());
    res.sendFile(
      path.resolve(__dirname, '../frontend', 'dist', 'index.html')
    );
  });
}
```

## Build your React frontend

The next thing you need to do is actually build your frontend. Run the following
command in your __frontend__ folder:

```sh
npm run build
```

This will pre-compile your frontend files and store them in a __frontend/dist__
folder.

Next, open your __frontend/.gitignore__ file. You will likely see `dist` listed
as one of the folders `git` should ignore. Folders with build files are
typically gitignored because they can be large, they can include sensitive
information like API keys, and the included files can always be regenerated
locally. Now, however, you need those files pushed to GitHub so you can access
them in deployment. Accordingly, add a `!` before `dist` in your frontend
__.gitignore__ (or add the whole phrase if `dist` does not already appear):

```plaintext
!dist
```

The `!` cancels the following pattern from being gitignored. This ensures that
your frontend __dist__ folder will be available for pushes to GitHub even if
other parent directories have __.gitignore__ files that would exclude it.

**Tip:** It's worth noting that **you must ALWAYS run `npm run build` in your
__frontend__ folder and commit those changes before pushing to GitHub /
deploying.** To help with this, you can add the `--watch` flag to the build
script in your __frontend/package.json__:

```json
  "build": "vite build --watch"
```

If you add this flag, then running `npm run build` will cause Vite to watch your
files and update the build anytime an included file changes. It's basically a
way of keeping the `build` task running. (You'll want to run this in a separate
terminal session.) Even with this flag, however, **you must still remember to
run `npm run build` at least once to start the watch.**

That's it! Your codebase should be ready. Go ahead and **commit your changes.**

## Phase 2: Deploying to Render

First, create a new web service. From your Render [Dashboard], click the `New +`
button on the top right. In the resulting dropdown menu, select `Web Service`.

If you have not already connected your GitHub account, click `Connect account`
and follow the steps to connect it. Once you have connected your GitHub account,
you should see a list of your available repos. Select the repo with your app.

Fill in the `Name` field with the name of your app. For `Root Directory`,
specify `backend`. Select `Node` as the runtime `Environment` if it is not
already selected. Choose the `Region` closest to you. Leave the branch as
`main`.

Replace the string in the `Build Command` with `npm install && node
seeders/seeds.js`.  
Replace the string in the `Start Command` with `npm start`.  
Select the `Free` plan.

> **Note:** Because you specified `backend` as your `Root Directory`, the build
> and start commands will be run from inside your __backend__ directory.

Under `Add Environment Variables`, create keys of
  `MONGO_URI` and `SECRET_OR_KEY`. Copy in their respective values from your
  __.env__ file.

Finally, click `Create Web Service`. This will take you to a page where you can
see the console output as your app builds. If all goes well, you should
eventually see `Build successful` followed by `Deploying...`.

## Visit your site

The web address for your site will have the form
`https://your-app-name.onrender.com`--possibly with some extra letters after
your app's name to avoid conflicts--and appear under your app's name near the
top of its web service page. Once your server is running, click your app's
address and visit it live on the web!

**Note:** It can take a couple of minutes for your site to appear, **even after
Render reports that it is `Live`.** Just be patient and refresh every 30 seconds
or so until it appears.

Unless you changed the `Auto-Deploy` setting from `Yes`, your app should rebuild
and redeploy whenever you push changes to your `main` branch. If you ever need
to trigger a manual redeployment, click the blue `Manual Deploy` button at the
top of your app's Render page. Select `Clear build cache & deploy` for a clean
redeployment from scratch. You will be able to see the logs and confirm that
your redeployment is successful.

[Render.com]: https://render.com/
[render-signup]: https://dashboard.render.com/register
[Dashboard]: https://dashboard.render.com/
[render-node]: https://render.com/docs/node-version
