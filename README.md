Investigating the confusion about which tsconfig "target" value
is used for:
https://github.com/open-telemetry/opentelemetry-js/pull/5456#discussion_r1956763421

This repo attempts to be a stripped down equivalent to the tsconfig
structure we are attempting at https://github.com/open-telemetry/opentelemetry-js/pull/5456

- `packages/a` will use `"target": "es2022",` via extending "tsconfig.base.json" at the repo root
- `packages/old` will use `"target": "es2017",` by overriding those settings from "tsconfig.base.json"
- `./tsconfig.json` at the root is setup to have references to both of these packages

`npm run compile` runs `tsc --build tsconfig.json` at the root, as does
opentelemetry-js.git's "compile" script.

What we are checking is that the compilation of `packages/old` will *break*
if it attempts to use syntax beyond ES2017. For example, ES2019 String.prototype.trimStart is used in its [index.ts](./packages/old/src/index.ts).

```bash
% npm run compile

> repro-tsc-compilation-target-confusion@1.0.0 compile
> rm -rf build packages/*/build && tsc --build tsconfig.json

packages/old/src/index.ts:2:45 - error TS2550: Property 'trimStart' does not exist on type '" spaces all around "'. Do you need to change your target library? Try changing the 'lib' compiler option to 'es2019' or later.

2 console.log('packages/old: str is: %j', str.trimStart());
                                              ~~~~~~~~~


Found 1 error.

```


This is failing as we expect.

Q: So why does it *not fail* for David, as he said at:
https://github.com/open-telemetry/opentelemetry-js/pull/5456#discussion_r1960209865
