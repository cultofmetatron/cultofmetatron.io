---
Categories:
- Development
- elixir
Description: "setting up auth0 with phoenix"
Tags:
- Development
- elixir
date: 2017-03-01T00:19:51-05:00
menu: post
title: Auth0 on Phoenix - Basic Configuration
---

Today I'm documenting the process of setting up [Auth0](https://auth0.com/) and [webpack2](https://webpack.js.org/) on a newly created [Phoenix](http://www.phoenixframework.org/) project
<!--more--> 
## creating the project

Create the project by running the generator

```

Mix phoenix.new MyApp

```

> Do not use --no-brunch. We'll remove that manually.

## Webpack Setup

I'm basing the webpack on an earlier publication by [Matthew Lehner](http://matthewlehner.net/using-webpack-with-phoenix-and-elixir/). His writeup is excellent but I noticed a few breaking changes trying to get webpack2 to work.

The first step is to remove the brunch configuration.

```
rm brunch-config.js

```

Next we install Webpack2 as well as our presets.

```

npm install --save-dev webpack babel-core babel-loader 
npm install --save-dev babel-preset-es2015 babel-preset-es2016

```

> optionally, you can include

```
npm install --save-dev babel-preset-es2017
npm install --save-dev babel-preset-react
```

Next, create webpack.config.js at the top level of your application.

```javascript
let path = require('path');

module.exports = {
  entry: {
    app: path.join(__dirname, "web/static/js/app.js")
  },
  output: {
    path: path.join(__dirname, "priv/static/js"),
    filename: "[name].js"
  },
  resolve: {
    modules: [ "node_modules", __dirname + "/web/static/js" ]
  },
  module: {
    loaders: [{
      test: /\.js$/,
      exclude: /node_modules/,
      loader: "babel-loader",
      query: {
        //add presets as necessary.
        presets: ["es2015", "es2016"] 
      }
    }]
  }

};

```

With that, You want to then add a watch script to npm. I also reccomend adding an *"engines"* property to lock down our node version.

It should look something like this.

```json
{
  "repository": {},
  "license": "MIT",
  "scripts": {
    "watch": "webpack --watch-stdin --progress --color"
  },
  "engines": {
    "npm": "6.9.x"
  },
  "dependencies": {
    "phoenix": "file:../../deps/phoenix",
    "phoenix_html": "file:../../deps/phoenix_html"
  },
  "devDependencies": {
    "babel-core": "^6.23.1",
    "babel-loader": "^6.3.0",
    "babel-preset-es2015": "^6.22.0",
    "babel-preset-es2016": "^6.22.0",
    "babel-preset-es2017": "^6.22.0",
    "babel-preset-react": "^6.23.0",
    "webpack": "^2.2.1"
  }
}
```


Next we want to remove the brunch watcher and add a webpack command to config/dev.exs

```elixir

config :web, MyApp,
  http: [port: 4000],
  debug_errors: true,
  code_reloader: true,
  check_origin: false,
  watchers: [
    npm: ["run", "watch", cd: Path.expand("../", __DIR__)]    
  ]

```

> The cd: Path.expand("../", __DIR__) ensures that our watcher will be run from the path of the root of our application. This will be very important if we run our phoenix app in an umbrella project.

## Auth0 Setup

[Auth0](https://auth0.com/) is an authentication as a service platform for secure login and user management. I'm using it for all but a few future projects because its secure, supports multitenancy and multifactor login as well as built in social integration to google, facebook, twitter, linkedin and github without me having to put in any extra effort. 

In This post, I'm going to cover a login through auth0's server. In a future post, I'll cover doing an auth0 oauth trhough our application server allowing us to setup sessions through guardian. For now, I will demonstrate how to just get the jwt token.

*auth0-lock* is the js file that contains their login widget. It can be installed via npm.

```

npm install --save auth0-lock

```

Now in your *app.js* add in the following code

```
import 'phoenix_html'
import socket from "./socket"

import Auth0Lock from 'auth0-lock';
let credentials = {
  domain: $YOUR_AUTH0_DOMAIN,
  clientId: $YOUR_CLIENT_ID
};

const cid = credentials.clientId;
const domain = credentials.domain;

const lock = new Auth0Lock(cid, domain);

lock.show();

lock.on("authenticated", function(authResult) {
  lock.getProfile(authResult.idToken, function(error, profile) {
    if (error) {
      // Handle error
      return;
    }
    localStorage.setItem('id_token', authResult.idToken);
    // Display user information
    console.log(profile)
    console.log('your jwt token is', authResult.idToken)
  });
});

```

Run `mix phoenix.server` and you should now see auth0 load up when you visit *localhost:4000* in your browser.


