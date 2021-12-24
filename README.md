<img src="https://sdkcon78221.crestron.com/downloads/ShowcaseApp/img/ch5-logo-dev-20181017-03.svg" width="40%"> <img src="https://svelte.dev/svelte-logo-horizontal.svg" width ="40%">

# Cres-Svelte-Tron

This is a un-offical project template and build pipeline for [Crestron CH-5](https://sdkcon78221.crestron.com/sdk/Crestron_HTML5UI/Content/Topics/Home.htm) apps using the [Svelte](https://svelte.dev) framework, [Rollup](https://rollupjs.org/guide/en/), and [Eruda](https://github.com/liriliri/eruda). It lives at https://github.com/purebordem/Cres-Svelte-Tron.

# Table of Contents
* [Getting Started](#getting-started)
* [Deploying to a Touch Panel](#deploying-to-a-touch-panel)
* [NPM Scripts](#npm-scripts)
* [Node CH5-Run](#node-ch5-run)
* [Mobile Console](#mobile-console)
* [Xpanel Support](#xpanel-support)
* [Rollup Config](#rollup-config)
* [Known Issues](#known-issues)
* [FAQ](#faq)
* [Release Notes](#release-notes)
---

## Getting started

If you do not already have it, install [Node.js](https://nodejs.org) and [Git](https://git-scm.com/)

* *Note - Cres-Svelte-Tron has only been tested with the latest LTS version*

Using a shell, navigate to a location on your drive where you wish to install and clone this repository...
```bash
git clone https://github.com/purebordem/Cres-Svelte-Tron
```

Next, inside the cloned repository, install the required dependencies...
```bash
npm install
```

Verify the installation by first running...
```bash
npm run dev
```

Navigate to [localhost:5000](http://localhost:5000). You should see the test app running locally on your machine. Use `Ctrl+C` inside the shell to end the dev session.


## Deploying to a Touch Panel
*Note - Only specific Crestron touch panels support HTML/JS/CSS based GUIs. The firmware must also be up to date.*

There are several ways to build and deploy to a Crestron touch panel. The easist is to use the `ch5-dev` script. This will auto-build the app, package it using [@Crestron/ch5-utlities](https://www.npmjs.com/package/@crestron/ch5-utilities), and deploy it to a specified touch panel. This script will also provide live reloading, building, and deployment. The app will also be accessible on [localhost:5000](http://localhost:5000).

First, go to the `package.json` file and change the host name in the following scripts to your panel's IP address...

```bash
"ch5-runDev": "node ch5-run --host XXX.XXX.XXX.XXX --dev",
"ch5-build": "node ch5-run --host XXX.XXX.XXX.XXX",
```

Next run the script...
```bash
npm run ch5-dev
```

*Note - This may take some time. Crestron's CrComLib, which communicates with the AV processor, adds considerable build time. If strictly working on the GUI and not integration, it is recommended to comment out `import CrComLib from '@crestron/ch5-crcomlib/build_bundles/cjs/cr-com-lib.js'` in the `src/App.svelte` file to speed up dev time*

If you are working with a touch panel with authentication enabled, you will want to add the `--user` and `--pass` flags to the scripts. The prompt feature of the CH5-CLI is not used since it would require manual entry everytime the page reloads, defeating the purpose of live reload.

## NPM Scripts
The following scripts are available by default...

### Build
This builds the app
```bash
npm run build
```

### Dev
This builds the app, in dev mode, with live reload, via [localhost:5000](http://localhost:5000)
```bash
npm run dev
```

### Start
This will deploy the app locally via [localhost:5000](http://localhost:5000) and your current IP address. Good for testing tablets, phones, etc.
```bash
npm run start
```

### CH5-Build
This builds the app, and deploys it to the touch panel at the specified in the `package.json`
```bash
npm run ch5-build
```

### CH5-Dev
This builds the app, in dev mode, with live reload, and deploys it to the touch panel at the specified in the `package.json`
```bash
npm run ch5-dev
```

## Node CH5-Run
This is direct usage of the `ch5-run.js` script used in other scripts. It provides a similar abstraction to the [CH5-Utilities-CLI](https://www.npmjs.com/package/@crestron/ch5-utilities-cli). Running directly will build the app and attempt to archive or deploy the project for CH5.
```bash
node ch5-run
```

```bash
Available flags...
	--name
    		Default Value: 'ch5-svelte'
	--path
    		Default Value: 'public'
	--output
    		Default Value: 'CH5-Build'
  	--host
    		Default Value: undefined
	--user
    		Default Value: undefined
	--pass
    		Default Value: undefined	
	--sftp
   		Default Value: 'display'
	--type
    		Default Value: 'touchscreen'
	--dev
    		Default Value: false
```

Refer to [@Crestron/ch5-utlities](https://www.npmjs.com/package/@crestron/ch5-utilities) for possible values.

If a value is not provided for a flag, the default value will be used. In the case of `--host`, undefined will cause the script to only archive the app as a `.ch5z` file and not try to deploy the archive.

## Mobile Console
Mobile browsers do not support a dev console like their PC counter-parts. Crestron's touch panels are running an Android version of Chromium and have the same issue. This can make checking for console issues on the touch panel tricky.

To get around this issue, Cres-Svelte-Tron uses [Eruda](https://github.com/liriliri/eruda) to create a close approximation. To use this feature, your `App.svelte` must include this following in the `<script` tag...
```bash
import * as eruda from 'eruda';
eruda.init();
```
Rollup will automatically remove this when building for production (`npm run ch5-build` or `node ch5-run` with no `--dev` flag).

## XPanel Support
Crestron recently opened up support CH5 to work as an XPanel via the [@crestron/ch5-webxpanel](https://www.npmjs.com/package/@crestron/ch5-webxpanel) package. You will also need an active [SW-Mobility License](https://www.crestron.com/Products/Control-Hardware-Software/Software/Licensing/SW-MOBILITY) for this to work. A simple use-case has now been included in the demo project. 

When working with `@crestron/ch5-webxpanel`, you must run import this package before importing the CrComLib. You must also insure the CrComLib is bound to the window since the ch5-webxpanel expects it to be globally available. Below is a basic implementatiom...
```bash
//import a library like CH5 CrComLib or the WebXpanel
//Note WebXPanel MUST be loaded first per Crestron if used
import * as WebXPanel from '@crestron/ch5-webxpanel/dist/cjs/index.js'
import CrComLib from '@crestron/ch5-crcomlib/build_bundles/cjs/cr-com-lib'

//attach required CrComLib functions so Svelte can communicate with CH5
window.CrComLib = CrComLib
window.bridgeReceiveIntegerFromNative = CrComLib.bridgeReceiveIntegerFromNative
window.bridgeReceiveBooleanFromNative = CrComLib.bridgeReceiveBooleanFromNative
window.bridgeReceiveStringFromNative = CrComLib.bridgeReceiveStringFromNative
window.bridgeReceiveObjectFromNative = CrComLib.bridgeReceiveObjectFromNative
```
More details about this package, the configuration options, and available methods, can be found at [Crestron's developer website](https://www.crestron.com/developer).

## Rollup Config
Cres-Svelte-Tron uses Rollup as the app bundler since it is the default used by the [Svelte Template](https://github.com/sveltejs/template). Changes made to `rollup.config.js` will alter how Rollup bundles the project. You can add support for other features such as PostCSS, SASS, and others here. The `production` variable is used to tell Rollup what to do for production builds vs. dev builds. Some useful Rollup plugins are included by default in Cres-Svelte-Tron.

### Limitations
The version of Chromium currently running on Crestron touch panels does not appear to support ES or CJS modules. This means Rollup must be set to use IIFE as an output format. Because of this format limitation, items such as dynamic imports and code splitting are not currently available.

## Known Issues
### Previous version of app sent to touch panel (always one revision behind)
Currently CH5-Utlities, responsible for creaing the `.ch5z` file, grabs the previously compiled files...despite the fact the files are overwritten (still figuring that one out...)
* Workaround - Rebuild the app again. If using live reload, edit a file and re-save.

### The dev console on the touch panel keeps saying 'null'
The live reload feature uses websockets on [localhost:35729](http://localhost:35729) to check for new updates. The touch panel however is not running a server so there is no [localhost](http://localhost) to communicate back to.

## FAQ
### Why Cres-Svelte-Tron?
Because [Svelte is awesome](https://svelte.dev/tutorial/basics)

### Ok, why Svelte?
It is a compiled framework unlike Angular, React, and Vue. This means it is generally smaller and faster. It also tries to stay closer to regular HTML/JS/CSS without completely re-inventing how to develop for the web. Think of it as adding language features and taking away some of the annoying parts. You get the benefit of two-way binding, reactivity, scoped styles, if/else for HTML, and a bunch more.

### Who should use Cres-Svelte-Tron?
Everyone! More specifically, if you find yourself needing more flexibility than the regular CH5 components provide but find CH5-Angular confusing, give Cres-Svelte-Tron a shot.

### How do I actually interface with the AV processor though?
You need to use the `CrComLib` from Crestron. Make sure you have `import CrComLib from '@crestron/ch5-crcomlib/build_bundles/cjs/cr-com-lib.js'` somewhere in your project. See the CH5-Utility-Functions section [here](https://sdkcon78221.crestron.com/downloads/ShowcaseApp/utility-functions/utility-subscribe-signal-script.html)

### The demo app sucks, I am not impressed
It is a naive implementation to show some basic concepts of how to integrate Svelte with CH5. There better ways to do several things in the demo so do not take it as gospel. It also is probably broke in a few places because the SIMPL side is very very bare bones and quickly thrown together.

### Where can I learn more about the CH5 side of things?
Visit Crestron's developer portal at [crestron.com/developer](https://www.crestron.com/developer). Just keep in mind most of the content in the CH5 side is focused around their CH5-Component framework. However, there are still general insights for integrating other frameworks like Svelte, Angular, React, and others.

## Release Notes
* v1.0.1
  * Updated all packages and implemented Github security fixes
  * Updated rollup config and ch5-run script to support latest version of dependencies
  * Added basic demo project along with SIMPL files for 3-Series processors
  * Implemented example for Web XPanel support
* v1.0.0
  * Initial release
