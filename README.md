I think I have found a bug in the Rescript compiler output.  I'm trying to
build bs-express as a dependency using es6 modules and the `.mjs` extension.
This is an old dependency, but it worked in commonjs format and is failing
under these conditions.


# To repro

`nvm install node 16.2.0`
`yarn install`

`yarn build`
`node src/demo.mjs`

Fails with:

```
file:///Users/dustyphillips/Desktop/Code/bsexpress-repro/node_modules/bs-express/src/Express.mjs:697
  return Express();
         ^

TypeError: Express is not a function
    at Module.express (file:///Users/dustyphillips/Desktop/Code/bsexpress-repro/node_modules/bs-express/src/Express.mjs:697:10)
    at file:///Users/dustyphillips/Desktop/Code/bsexpress-repro/src/demo.mjs:5:19
    at ModuleJob.run (node:internal/modules/esm/module_job:175:25)
    at async Loader.import (node:internal/modules/esm/loader:178:24)
    at async Object.loadESM (node:internal/process/esm_loader:68:5)
```

edit the generated `node_modules/bs-express/src/Express.mjs` and change line 7 from:

`import * as Express from "express";`

to

`import Express from "express";`

Now it works:

`yarn build`
`node src/demo.mjs` succeeds

# Notes on the repo

This repo was initialized with `npx rescript init`. You can view the
commit history to see exactly what changes I made, but it amounted to:

* change `bsconfig.json` to have `"module": "es6"` and `"suffix": ".mjs"`
* Add `"type": "module"` to `package.json`
* `yarn add bs-express`
* Add `bs-express` to the dependencies in `bsconfig.json`
* Change `demo.res` to construct an express 

I think that recript is either generating the import from "express"
incorrectly, or is calling it incorrectly, as evidenced by changing the
import in the generated code.

