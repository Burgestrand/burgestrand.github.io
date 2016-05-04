---
title: Peeling layers off from Heroku deployments of Phoenix
date: 4 May 2016 16:00:00 GMT+2
categories: [elixir, heroku, deployment]
---

[Heroku Buildpack: Phoenix Static](https://github.com/gjaldon/heroku-buildpack-phoenix-static) can be used to
compile static assets when deploying Phoenix to Heroku. I used it a while, until I realized:

1. I'm not using brunch, I use webpack. Thank you, [Matthew Lehner](http://matthewlehner.net/using-webpack-with-phoenix-and-elixir/) for saving me a lot of time setting this up.
1. I'm not using bower, because webpack.
1. [Heroku supports multiple buildpacks for the same application](https://devcenter.heroku.com/articles/using-multiple-buildpacks-for-an-app), so why
   does Phoenix Static install Node and NPM on it's own?

Question no. 3 got me going. Could I use only the Elixir and Node buildpacks, and thus remove
the Phoenix Static buildpack and the assumption that I'm using brunch from my set-up, all at the same time?

Yes! It did require [a pull request to allow for the Elixir buildpack to be used in multi-buildpack setups](https://github.com/HashNuke/heroku-buildpack-elixir/pull/74),
but that was a nice thing to be done anyway. I still use two buildpacks: [Elixir & Mix](https://github.com/HashNuke/heroku-buildpack-elixir) together with
[Node & NPM](https://github.com/heroku/heroku-buildpack-nodejs) (which, by the way, is maintained by Heroku), in that order.

Once that was configured, I used the `postbuild` hook from the Node buildpack to compile my assets. It looks like this:

```json
{
  "scripts": {
    "start": "webpack --watch-stdin --progress --color",
    "compile": "webpack --optimize-minimize",
    "heroku-postbuild": "npm run compile && mix phoenix.digest"
  }
}
```

It feels like a cleaner separation of concerns. In my previous setup, Phoenix Static was not only in charge of installing Node, NPM, and 
Bower, but also for compiling my assets using Brunch and digesting them with Phoenix.

Now the Elixir buildpack is in charge of installing Elixir, the Node buildpack is in charge of installing Node, and I am in charge of
compiling my assets.

Oh, final caveat, you might want a `Procfile` to communicate to Heroku how your application should be run.
I like to have this anyway, again because of explicitness, but because the Node buildpack is the last in
the pipeline I think it might actually be required.
