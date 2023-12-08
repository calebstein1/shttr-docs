# Shttr CLI

## About Shttr CLI

Shttr CLI is the main command line interface for managing Shttr apps.
It handles creating apps, creating and removing pages, managing build-time and runtime NodeJS dependencies, running the development server, and debugging and deploying apps.
It also manages your installed Shell on the Shttr runtime.

## Creating Apps

```Shell
shttr n
shttr new
shttr n app_name
shttr n app_name --bootstrap
shttr n app_name --tailwind
shttr n app_name --empty
```

Running `shttr n`, or `shttr new` will create a new Shell on the Shttr project.
It takes the app name as an argument, and optionally takes a framework specifier after the app name.
By default, running `shttr n test_app` will create a Shttr project called "test_app" populated with the default test site.
This is the recommended way to get started, as you can immediately run the test server to confirm that it's all working.
Optionally, you can run `shttr n test_app --boostrap` or `shttr n test_app --tailwind` to generate empty projects preconfigured to use Bootstrap or TailwindCSS respectively.
You also have the option to run `shttr n test_app --empty` to generate an entirely empty project with no pages and no preconfigured frontend frameworks.
Shttr CLI also initializes all new apps as git repos.

## Generating Assets

``Shell
shttr g page page_name
shttr generate page page_name
``

You can use `shttr g` or `shttr generate` to generate assets for your app.
Currently, there is only one supported generator, the page generator.
You would run `shttr g page [page name]` to generate a new page with your selected name.
This creates the proper entrypoint script in `cgi-bin`, a barebones controller, a blank model, and a barebones view for your page.
If Shttr CLI finds a nav.html file in your shared views, that file will be included in your new page's view underneath app.html.

## Deleting Assets

```Shell
shttr r page page_name
shttr remove page page_name
```

The opposite of the generator, `shttr r` or `shttr remove` deletes assets from your app.
Much like the generator, it only supports pages at the moment.
Running `shttr r page [page name]` would remove the entrypoint, controller, model, and view associated with the page name supplied.

## Compiling Static Assets

```Shell
shttr c
shttr compile
```

Running `shttr c` or `shttr compile` will compile all stylesheets down to one css file named with a checksum that will get automatically loaded by each page in your app.
It also ensures that all runtime NodeJS dependencies are installed and updated correctly.
Because Shttr supports sass as well as css, and because the sass compiler is slow, the compile step is not run by default on every deploy or server operation.
You should really only run the compile step if you've recently updated your stylesheets.

## Managing Runtime NodeJS Dependencies

```Shell
shttr npm i
shttr npm i node_package
shttr npm uninstall node_package
```

Your app's runtime NodeJS dependencies are handled by using `shttr npm`.
This supports any standard npm command, and ensures that all NodeJS packages are installed into the proper app directory.
For example, `shttr npm i @domchristie/turn` would install the Turn javascript library into your app as a runtime dependency, and `shttr npm uninstall @domchristie/turn` would remove it.

## Running the Server

```Shell
shttr s
shttr server
shttr s --production
```

You can use `shttr s` or `shttr server` to run the Shttr server.
This will build a Docker image and then run the integrated Apache web server, which can be accessed at localhost:3000.
This approach ensures consistency between the development and production environment.
The server optionally takes `--production` as a command line flag, which will run the server in production mode.
At the moment, this has no effect, but once Shttr SQL is released, this switch will allow for different development and production databases to be used.
Because of the containerized nature of the server, it does not support live reloading.
If you make a change to your app files, you will need to build a new image and relaunch the server to see the changes reflected.

## Debugging Pages

```Shell
shttr debug page_name
```

The Shttr debugger can be run using `shttr debug [page name]`.
This will create an environment that mimics the CGI environment of the server and runs the page scripts, ultimately outputting the resultant HTML as plain text to teh console.
The debugger is also looking for any instances of `dbg_break` in your scripts.
Any occurrence of `dbg_break` will pause execution at that point and drop you into a shell where any valid shell command is available to you to run at that exact point in the execution of the script.
This is useful for inspecting variables at various points in the page's execution, or testing how different functions react to different values without having to edit your source files.
Running `continue` at the debugger shell prompt will resume execution.
You can have as many instances of `dbg_break` as you like, the debugger will simply pause at each one of them as it gets to them.

## Deploying Pages

```Shell
shttr d
shttr deploy
shttr d -c
```

In order to deploy your Shttr apps, you'll run `shttr d` or `shttr deploy`.
Adding the `-c` flag to the deploy command will run the compile step prior to building the image and deploying.
Shttr CLI will read the values in your app's `config.sh` file to determine how to deploy the app.
Currently, there is only one supported deployment target: systemd.
This target will deploy your app over SSH to a remote server with Docker installed and register itself as a SystemD service on that server.
You'll then need to set up a web server and a reverse proxy to localhost:3000 on the server.
This deployment target is very convenient for users of VPS services, since as long as your server already has Docker installed and your reverse proxy is set up, you can deploy your app with a single command.
There is currently an AWS Elastic Beanstalk deployment under development.

## Updating the System's Shttr Runtime

```Shell
shttr su
```

Running `shttr su` will update your system's current installed Shell on the Shttr runtime by pulling the latest version from the Github repo.
This does not affect any already created apps, but the new version will be used for any new apps that are created after updating.

## Updating an App's Shttr Runtime

```Shell
shttr u
shttr update
```

In order to update the Shell on the Shttr runtime for an already created app, you'd run `shttr u` or `shttr update` within that app's directory.
Be careful doing this, as it is a one-way process, and since Shell on the Shttr is still under heavy development, breaking changes are to be expected.
Make sure you understand what's changed in the new Shell on the Shttr runtime and why you need to update before running this command.