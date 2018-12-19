
[![Maintained by Gruntwork.io](https://img.shields.io/badge/maintained%20by-gruntwork.io-%235849a6.svg)](https://gruntwork.io/?ref=repo_gruntwork-cli)
[![GoDoc](https://godoc.org/github.com/gruntwork-io/gruntwork-cli?status.svg)](https://godoc.org/github.com/gruntwork-io/gruntwork-cli)

# Gruntwork CLI

This repo contains common libraries and helpers we can use to build Gruntwork CLI tools.

## Packages

This repo contains the following packages:

* collections
* entrypoint
* errors
* files
* logging
* shell

Each of these packages is described below.

### collections

This package contains useful helper methods for working with collections such as lists and maps, as Go has very few of
these built-in.

### entrypoint

Most Gruntwork CLI apps should use this package to run their app, as it takes
care of common tasks such as setting the proper exit code, rendering stack
traces, handling panics, and rendering help text in a standard format. Note
that this package assumes you are using
[urfave/cli](https://github.com/urfave/cli), which is currently our library of
choice for CLI apps.

Here is the typical usage pattern in `main.go`:

```go
package main

import (
        "github.com/urfave/cli"
        "github.com/gruntwork-io/gruntwork-cli/entrypoint"
)	

// This variable is set at build time using -ldflags parameters. For example, we typically set this flag in circle.yml
// to the latest Git tag when building our Go apps:
//
// build-go-binaries --app-name my-app --dest-path bin --ld-flags "-X main.VERSION=$CIRCLE_TAG"
//
// For more info, see: http://stackoverflow.com/a/11355611/483528
var VERSION string

func main() {
      // Create a new CLI app. This will return a urfave/cli App with some
      // common initialization.
      app := entrypoint.NewApp()
    
      app.Name = "my-app"
      app.Author = "Gruntwork <www.gruntwork.io>"
      
      // Set the version number from your app from the VERSION variable that is passed in at build time
      app.Version = VERSION
      
      app.Action = func(cliContext *cli.Context) error { 
        // ( fill in your app details)
        return nil
      }

      // Run your app using the entrypoint package, which will take care of exit codes, stack traces, and panics
      entrypoint.RunApp(app)
}
```

### errors

In our CLI apps, we should try to ensure that:

1. Every error has a stacktrace. This makes debugging easier.
1. Every error generated by our own code (as opposed to errors from Go built-in functions or errors from 3rd party
   libraries) has a custom type. This makes error handling more precise, as we can decide to handle different types of
   errors differently.

To accomplish these two goals, we have created an `errors` package that has several helper methods, such as
`errors.WithStackTrace(err error)`, which wraps the given `error` in an Error object that contains a stacktrace. Under
the hood, the `errors` package is using the [go-errors](https://github.com/go-errors/errors) library, but this may
change in the future, so the rest of the code should not depend on `go-errors` directly.

Here is how the `errors` package should be used:

1. Any time you want to create your own error, create a custom type for it, and when instantiating that type, wrap it
   with a call to `errors.WithStackTrace`. That way, any time you call a method defined in our own code, you know the 
   error it returns already has a stacktrace and you don't have to wrap it yourself.
1. Any time you get back an error object from a function built into Go or a 3rd party library, immediately wrap it with
   `errors.WithStackTrace`. This gives us a stacktrace as close to the source as possible.
1. If you need to get back the underlying error, you can use the `errors.IsError` and `errors.Unwrap` functions.
1. If you want to return an error that forces a specific exit code, wrap it with `errors.ErrorWithExitCode`.

Note that `entrypoint.RunApp` takes care of showing stack traces and handling exit codes.

### files

This package has a number of helpers for working with files and file paths, including one-liners for checking if a 
given path is a file or a directory, reading a file as a string, and building relative and canonical file paths.

### logging

This package contains utilities for logging from our CLI apps. Instead of using Go's built-in logging library, we are 
using [logrus](github.com/sirupsen/logrus), as it supports log levels (INFO, WARN, DEBUG, etc), structured logging 
(making key=value pairs easier to parse), log formatting (including text and JSON), hooks to connect logging to a 
variety of external systems (e.g. syslog, airbrake, papertrail), and even hooks for automated testing.
 
To get a Logger, call the `logging.GetLogger` method:
 
```go
logger := logging.GetLogger("my-app")
logger.Info("Something happened!")
```

To change the logging level globally, call the `SetGlobalLogLevel` function:

```go
logging.SetGlobalLogLevel(logrus.DebugLevel)
```

Note that this ONLY affects loggers created using the `GetLogger` function **AFTER** you call `SetGlobalLogLevel`, so
you need to call this as early in the life of your CLI app as possible!

### shell

This package contains two types of helpers:

* `cmd.go`: This file contains helpers for running shell commands.
* `prompt.go`: This file contains helpers for prompting the user for input (e.g. yes/no).

## Running tests

```
go test -v ./...
```
