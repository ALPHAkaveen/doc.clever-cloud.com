## Configure your Haskell application

### Mandatory configuration

Be sure that:

* you have pushed in **master branch**
* you listen on **port 8080**
* you have one and only one binary target in your [`.cabal` file](#dependencies)
* your project has a `stack.yaml` file

The steps ran in order are:

 - `stack setup`
 - `stack install --only-dependencies`
 - `stack install`
 - `./<path>/my-exe` (or the contents of `CC_RUN_COMMAND`)

The executable built by `stack build` (or the command you specify) must start a web server listening on `0.0.0.0:8080`.

For instance, a minimal [scotty](https://hackage.haskell.org/package/scotty) application can look like this:

```haskell
{-# LANGUAGE OverloadedStrings #-}

import Web.Scotty

main = scotty 8080 $ do
  get "/" $ do
    html $ "Hello world"
```

### Dependencies

Make sure to list all your dependencies in your `.cabal` file. For the example above, you need:

```txt
  build-depends:       base
                     , scotty
```

### Cached dependencies

#### Enabling dependencies caching

You can enable dependencies caching by adding the `CC_CACHE_DEPENDENCIES=true` [environment variable](#setting-up-environment-variables-on-clever-cloud) in your application. It is enabled by default only for rust and haskell applications.

#### Disabling dependencies caching

You can disable dependencies caching completely by removing the `CC_CACHE_DEPENDENCIES` environment variable from the Clever Cloud console, in the **Environment variables** menu of your application. Or by setting it to `CC_CACHE_DEPENDENCIES=false`
To fully remove cached dependencies, you have to rebuild your application from scratch. You can select "rebuild and restart" from the Clever Cloud console or launch `clever restart --without-cache` with the Clever Tools CLI.

### Custom run command

If you need to run a custom command (or just pass options to the binary built
by stack), you can specify it through the `CC_RUN_COMMAND` environment variable.

For instance, for a hakyll website, you can put

```bash
CC_RUN_COMMAND="~./local/bin/site server --host 0.0.0.0 --port 8080"
```

To access environment variables from your code, just get them from the environment with `getEnv :: String -> IO String` in `System.Environment`.

If some environment variables are critical to your application, here is an approach you can use:

```haskell
{-# LANGUAGE OverloadedStrings #-}
module Main where

import Data.Text.Lazy (pack)
import System.Environment (getEnv)
import Web.Scotty (get, html, scotty)

envVar = getEnv "MY_VAR"

main = do
  myVar <- envVar
  scotty 8080 $ do
    get "/" $ do
      html . pack $ "Hello world " ++ myVar
```

This loads the environment variable in your `main` function and tests it. This way, the application will refuse to start with an helpful error message if `MY_VAR` is not defined.