# Anatomy of a Shttr App

## Shttr Project Base Directory Structure

Upon creating a new Shttr project, you'll end up with a directory structure that approximates what follows.
This section will go into detail about what each directory or file does.

```
├── Dockerfile.dev
├── Dockerfile.prod
├── README.md
├── cgi-bin
│   ├── about.sh
│   └── index.sh
├── config.sh
├── node_modules
├── package-lock.json
├── package.json
└── shttr
    ├── app
    │   ├── assets
    │   │   ├── images
    │   │   │   ├── github-mark-white.png
    │   │   │   └── github-mark.png
    │   │   ├── package.json
    │   │   ├── public
    │   │   │   └── style-c338dd97768b97ae3a7e1bf8d1f39cd601b15f5fdd09a35564b153029f675b66.css
    │   │   ├── scripts
    │   │   │   └── vendored
    │   │   │       └── turbo.js
    │   │   └── stylesheets
    │   │       └── vendored
    │   │           └── pico.scss
    │   ├── controllers
    │   │   ├── about.sh
    │   │   └── index.sh
    │   ├── globals.sh
    │   ├── models
    │   │   ├── about.sh
    │   │   └── index.sh
    │   └── views
    │       ├── about
    │       │   └── index.md
    │       ├── about.sh
    │       ├── index
    │       │   └── index.md
    │       ├── index.sh
    │       └── shared
    │           ├── app.html
    │           ├── btm.html
    │           └── nav.html
    ├── bin
    └── lib
```

## Dockerfile

A Shttr project contains two Dockerfiles: `Dockerfile.dev` and `Dockerfile.prod`.
These seperate Dockerfiles are used to build the development and production images of you project respectively.
The difference between the two Dockerfiles is that Dockerfile.dev bind-mounts the `cgi-bin` and `shttr` directories in the image which allows for live-reloading changes without having to restart the server, whereas Dockerfile.prod copies the app files into the image which creates an image that is read-only as long as it's running.
Managing these two Dockerfiles requires no intervention on the part of the app developer.
When starting the server, ShttrCLI will automatically symlink `Dockerfile` to `Dockerfile.dev` or `Dockerfile.prod` depending on whether or not `--production` was passed, and in deployment, `Dockerfile.prod` will be pushed to the deployment target as `Dockerfile` automatically.

## cgi-bin

The `cgi-bin` directory is one of the key app directories, though it doesn't typically require any user-intervention.
Shttr does not utilize a router like many other popular frameworks, instead Shttr using entrypoint scripts that live in `cgi-bin`.
Using `shttr g page` will automatically create the proper entrypoint script for your new page, and these entrypoints can be navigated to in your views using `/cgi-bin/page.sh` as the URL.
Each entrypoint script should be identical; their job is simply to set up certain key environment variables that need to exist prior to executing the main Shttr script, they then pass execution off to the main Shttr script.
It's very rare that any of these scripts should need to be modified.

## config.sh

The `config.sh` file contains project-specific configuration options used by ShttrCLI.
At the moment, it contains deployment options and supports the following two parameters: `DEPLOYMENT_TARGET` and `SSH_ADDRESS`.
`DEPLOYMENT_TARGET` determines the app's deployment target, currently there is one supported value: `systemd`, which deploys the production Docker image to a web server as a SystemD service.
`SSH_ADDRESS` sets the username and address of the server where the Docker image and SystemD service is to be deployed.
It takes the standard format `user@example.com`.

## node_modules

The `node_modules` directory, as well as the `package.json` and `package-lock.json` files in the root of your project directory contain build-time NodeJS package dependencies which are used to build your app but do not need to be bundled with it in deployment.
These build dependencies are managed by ShttrCLI and should not be modified.

## shttr/app/assets

This directory contains static assets to be used by your Shttr pages.
The directory can be accessed by your views using the environment variable `SHTTR_ASSETS`.

### images

The `images` directory contains images that can be used on your pages.
The src URL in your img tag would look like `${SHTTR_ASSETS}/images/image.png`.

### node_packages, package.json, package-lock.json

These directories and files contain NodeJS runtime packages for your project.
These _will_ be bundled with your app in deployment.
The `shttr npm` command controls packages installed here, and any package installed and imported in a page can be accessed using the variable `node_package-name`.
The `node_package-name` variable points to the package's installation directory, so for example, if we'd installed the Turn library using `shttr npm i @domchristie/turn` and we'd imported the library using `use @domchristie/turn`, we'd have access to the variable `node_turn` which would expand to `/app/assets/node_modules/@domchristie/turn`.
This variable would be used within our pages or in an importmap to access the Node package.

### public

This directory contains the compiled stylesheet which is used by the view pages.
When assets are compiled using either `shttr d -c` or `shttr c`, all stylesheets used by the project are compiled into one .css file named with a checksum digest.
A link to this file is then automatically injected into the head of each view by default.
Note that only one .css file can exist in this directory at a time, otherwise the automatic linking will fail.

### scripts

This directory contains Javascript files to be used by your views.
Any script in the `vendored` subdirectory will be injected into the head of each page automatically by default.
Any scripts outside `vendored` will need to be explicitly loaded into a script tag using `${SHTTR_ASSETS}/scripts/script.js` as the src URL.

### stylesheets

This directory contains the source stylesheets that are to be modified by the app developer.
All .css and .scss files within the `stylesheets` directory and any subdirectories will be compiled into one big .css file named with a checksum digest and placed under `public` for use by the running app.
This compilation is triggered by running `shttr d -c` or `shttr c`.

## shttr/app/controllers

This directory contains the controller script for each page.
A basic controller script will just set the page title, run the model, and then run the view script.
Controllers can get quite involved on complex pages, while the proper documentation is in progress, see the code for [calebstein.net](https://calebstein.net) [here](https://github.com/calebstein1/calebstein-web) for examples.

## shttr/app/globals.sh

This file is run on every page load in the app.
It's a good spot to declare any modules that will be needed by every page, or to do any global prep work based on saved cookies.

## shttr/app/models

The files in this directory correspond to files in the `controllers` directory.
Each controller runs its corresponding model in its environment, and while it's not strictly necessary to use the model script, it's there for teh sake of code organization.

## shttr/app/views

Each page view has two components: a script and a directory of partials.
The script uses the `esh` module and shell logic to render the HTML response to the client.
Each directory of partials contains component files, which can either be HTML files, or any file that Pandoc can convert to HTML.
These files do not require use of a specific templating language, but any environment variables in them will be replaced with their proper values upon rendering by the `esh` module.

## bin

This contains the main Shttr executable script.
After the entrypoint finishes, execution is handed to the `shttr` script here, which further sets up the environment for the controller to take over.
Do not modify files in this directory unless you really know what you're doing.

## lib

This directory contains the modules that are imported by pages with the `uae` command.
If you were to write your own module, you could stick it here and access it with `use`, but that's not really officially supported at the point so it's recommended to leave this directory alone.
Just accept the magic ;)