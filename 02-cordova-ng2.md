Overview
==========

Introducing Angular 2 into Cordova applications bring their own set of complexities.

Problem: Crypto libraries are stripped by angular-cli
-----------------------------------------------------

`Auth0` relies on `crypto-js` being present. Unfortunately, `angular-cli` strips it out. To overcome this, 

    ng eject
    
and update `webpack.config`:

     "node": {
      "fs": "empty",
      "global": true,
      "crypto": true, /* workaround:cordova */
      "tls": "empty",
      "net": "empty",
      "process": true,
      "module": false,
      "clearImmediate": false,
      "setImmediate": false
    },
    
Problem: Angular bootstrap should run after deviceready event  
--------------------------------------------------------------------

In `main.ts`, conditionally bootstrap after deviceready if cordova is detected.

    if (environment.production) {
      enableProdMode();
    }

    function bootstrap() {
      console.log(`bootstrap started`);

      let promise = platformBrowserDynamic().bootstrapModule(AppModule);

      /* workaround: Tried to find bootstrap code, but could not. 
      ** Specify either statically analyzable bootstrap code or 
      ** pass in an entryModule to the plugins options. 
      ** (split into separate lines to fool AOT)
      */
      promise.then(() => {
        console.log(`bootstrap completed`);
      }).catch(err => console.error(err));      
    }

    var isCordovaApp = !!window['cordova'];
    if (isCordovaApp) {
      document.addEventListener('deviceready', bootstrap, false);
    } else {
      bootstrap();
    }
    
Problem: Angular errors with "Tried to find bootstrap code, but could not. Specify either statically analyzable bootstrap code or...
--------

The angular compiler statically analyzes the application and 
tries to infer the bootstrap module. Unfortunately, it is buggy
and fails if you do this.

      platformBrowserDynamic().bootstrapModule(AppModule)
      .then(() => {
        console.log(`bootstrap completed`)
      })
      .catch(err => console.error(err));
      
You can workaround this by splitting the statement into separate lines.

      let promise = platformBrowserDynamic().bootstrapModule(AppModule);
      
      promise
      .then(() => {
        console.log(`bootstrap completed`)
      })
      .catch(err => console.error(err));
