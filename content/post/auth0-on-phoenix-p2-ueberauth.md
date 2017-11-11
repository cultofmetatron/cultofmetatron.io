---
Categories:
- Development
- GoLang
Description: ""
Tags:
- Development
- golang
date: 2017-03-06T21:21:25-05:00
menu: post
title: Auth0 on Phoenix Ueberauth
---

Previously I covered how to install auth0 on your browser.
There are limitations to a client based approach.
For one, you are limited strictly to jwt login. Furthermore,
you can't run execute any jobs to be performed on login.
If you want to enable cookie sessions or run any sorts of side
effect behavior to occur on login, you're going to need to setup
the oauth redirect to go through your phoenix server.

<!--more-->

## Ueberauth

[Ueberauth](https://github.com/ueberauth/ueberauth) is a set of middleware
for Phoenix that allows you to hook in oauth strategies for various identity
providers in a consistant way. Its kind of like Passport if you use nodejs.

First You want to install ueberauth and ueberauth-auth0.

You'll want to add the followign hex packages to your application.

> Mix.exs

```
defmodule LaVoyage.Web.Mixfile do
  use Mix.Project

  ...

  # Configuration for the OTP application.
  #
  # Type `mix help compile.app` for more information.
  def application do
    [mod: {LaVoyage.Web, []},
     applications: [
       ...
       :auth0_ex,
       :ueberauth,
       :oauth2
      ]
    ]
  end

  # Specifies which paths to compile per environment.
  defp elixirc_paths(:test), do: ["lib", "web", "test/support"]
  defp elixirc_paths(_),     do: ["lib", "web"]

  # Specifies your project dependencies.
  #
  # Type `mix help deps` for examples and options.
  defp deps do
  [
     ...
     {:auth0_ex, "~> 0.1"},
     {:ueberauth, "~> 0.4"},
     {:ueberauth_auth0, "~> 0.1"},
     {:oauth2, "~> 0.9"},
    ]
  end
end


```

Now just run `mix deps.get` and add the following to your `config/config.exs`

```
config :auth0_ex,
  domain: AUTH0_DOMAIN,
  mgmt_client_id: CLIENT_ID 
  mgmt_client_secret: CLIENT_SECRET
  http_opts: []

config :ueberauth, Ueberauth.Strategy.Auth0.OAuth,
  domain: AUTH0_DOMAIN,
  client_id: CLIENT_ID ,
  client_secret: CLIENT_SECRET

config :ueberauth, Ueberauth,
providers: [
  auth0: {Ueberauth.Strategy.Auth0, []}
]

config :oauth2,
  warn_missing_serializer: false,
  serializers: %{
    "application/vnd.api+json" => Poison,
    "application/json" => Poison
  }

```

Create a *SessionController* module in your controller.

Ueberauth gives you a few macros that generate controller endpoints that
call the `callback()` method. You want to add the following callback and request methods in
your SessionController.

```
defmodule SessionController do
  use LaVoyage.Web.Web, :controller
  plug Ueberauth

  def request(conn, _params) do
    render(conn, "login.html", callback_url: Ueberauth.Strategy.Helpers.callback_url(conn))
  end

  def callback(%Plug.Conn{assigns: %{ueberauth_failure: fails}} = conn, _params) do
    conn
    |> put_flash(:error, hd(fails.errors).message)
    |> render("login.html", callback_url: Ueberauth.Strategy.Helpers.callback_url(conn))
  end

  def callback(%Plug.Conn{assigns: %{ueberauth_auth: auth}} = conn, _params) do
    token = auth |> Map.get(:credentials) |> Map.get(:other) |> Map.get("id_token")
    conn
      |> put_session(:current_user, auth)
      |> put_session(:jwt_token, token)
      |> rediect(to: page_path(conn, :index))
  end
end
```

This callback with pass a Plug connection with an ueberauth auth object containing
all the details of the login request. from here we can perform any application
specific activity that may be necessary for your app. In this case, we get the
jwt from the auth object and put it into the user session along with user data
before issuing a redirect.


Now in auth0, you want to add your url to the list of allowed callbacks

![Alt text](/images/auth0_cb_urls.png)

Now we just need to add your route

```
scope "/auth", LaVoyage.Web do
  pipe_through :browser
  get  "/signin", SessionController, :request
  get  "/:provider/callback", SessionController, :callback
end
```

That should give us everything we need on the backend. Next we need to go back
and configure Auth0-lock on the frontend to redirect through out server

> app.js

```javascript
import Auth0Lock from 'auth0-lock';
import socket from "./socket"

import { cid, domain } from 'configs/auth0';

const lock = new Auth0Lock(cid, domain, {
  auth: {
    redirectUrl: "http://localhost:4000/auth/auth0/callback",
    responseType: 'code',
    params: {
      scope: 'openid email'
    }
  }
});

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
  });
});

```

Now you should be ready to go.


