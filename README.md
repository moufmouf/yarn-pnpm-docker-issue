# Yarn issue

I'm running with an issue with running "yarn run xxx" commands in Yarn Berry (3.3.0) + pnpm linker + Docker

This is a sample test case to reproduce the issue.

The issue is that if I run "yarn install" in the container, then "yarn run xxx" will run only inside the container (not outside).
Reversely, if I run "yarn install" out of the container, then "yarn run xxx" will run only outside the container (not inside).

In order to reproduce, clone this repository:

```
git clone https://github.com/moufmouf/yarn-pnpm-docker-issue.git
```

Then, run `yarn install` out of the container:

```
yarn install
```

The package.json contains "prettier", so you can run `yarn run prettier`.

This will work as expected.

Now, run `docker-compose up`. This will run `yarn run prettier` inside a NodeJS container.

You will see this error message:

```
Starting testyarn_test_1 ... done
Attaching to testyarn_test_1
test_1  | node:internal/modules/cjs/loader:998
test_1  |   throw err;
test_1  |   ^
test_1  | 
test_1  | Error: Cannot find module '/home/david/tmp/testyarn/node_modules/.store/prettier-npm-2.8.1-be60b51821/node_modules/prettier/bin-prettier.js'
test_1  |     at Module._resolveFilename (node:internal/modules/cjs/loader:995:15)
test_1  |     at Module._load (node:internal/modules/cjs/loader:841:27)
test_1  |     at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
test_1  |     at node:internal/main/run_main_module:23:47 {
test_1  |   code: 'MODULE_NOT_FOUND',
test_1  |   requireStack: []
test_1  | }
test_1  | 
test_1  | Node.js v18.12.0
testyarn_test_1 exited with code 1
```

Note that Node inside the container tries to find a module at `/home/david/tmp/testyarn/node_modules` which is my "host"
directory. The correct path inside the container would be `/usr/src/app/node_modules` but somehow, it seems Yarn
is keeping a reference of the host path somewhere. The strange thing is I can't find where....
