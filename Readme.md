# Problem

Typescript's [`typesVersions`](typesVersions) allows package to map generated `.d.ts` files produces by `tsc --emitDeclartions` so that users of the package will get all the typings out of the box.

However it seems that path substitution does not seems to work well with `main` field in `package.json`.
This repo illustartes the problem:

When original path `node_modules/bar/src` does not exists TS fails to resolve mapped path `node_modules/bar/dist/src/index.d.ts`.


```
tsc --noEmit --traceResolution
======== Resolving module 'bar' from '/Users/gozala/Projects/bug-tsc-double-substitution/src/main.ts'. ========
Explicitly specified module resolution kind: 'NodeJs'.
Loading module 'bar' from 'node_modules' folder, target file type 'TypeScript'.
Directory '/Users/gozala/Projects/bug-tsc-double-substitution/src/node_modules' does not exist, skipping all lookups in it.
Found 'package.json' at '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/package.json'.
'package.json' has a 'typesVersions' field with version-specific path mappings.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar.ts' does not exist.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar.tsx' does not exist.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar.d.ts' does not exist.
'package.json' does not have a 'typings' field.
'package.json' does not have a 'types' field.
'package.json' has 'main' field 'src/index.js' that references '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/src/index.js'.
'package.json' has a 'typesVersions' entry '*' that matches compiler version '4.0.3', looking for a pattern to match module name 'src/index.js'.
Module name 'src/index.js', matched pattern '*'.
Trying substitution 'dist/*', candidate module location: 'dist/src/index.js'.
Loading module as file / folder, candidate module location '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/index.js', target file type 'TypeScript'.
File name '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/index.js' has a '.js' extension - stripping it.
Directory '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/@types' does not exist, skipping all lookups in it.
File '/Users/gozala/Projects/node_modules/bar.ts' does not exist.
File '/Users/gozala/Projects/node_modules/bar.tsx' does not exist.
File '/Users/gozala/Projects/node_modules/bar.d.ts' does not exist.
Directory '/Users/gozala/Projects/node_modules/@types' does not exist, skipping all lookups in it.
Directory '/Users/gozala/node_modules' does not exist, skipping all lookups in it.
Directory '/Users/node_modules' does not exist, skipping all lookups in it.
Directory '/node_modules' does not exist, skipping all lookups in it.
Loading module 'bar' from 'node_modules' folder, target file type 'JavaScript'.
Directory '/Users/gozala/Projects/bug-tsc-double-substitution/src/node_modules' does not exist, skipping all lookups in it.
Found 'package.json' at '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/package.json'.
'package.json' has a 'typesVersions' field with version-specific path mappings.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar.js' does not exist.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar.jsx' does not exist.
'package.json' has 'main' field 'src/index.js' that references '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/src/index.js'.
'package.json' has a 'typesVersions' entry '*' that matches compiler version '4.0.3', looking for a pattern to match module name 'src/index.js'.
Module name 'src/index.js', matched pattern '*'.
Trying substitution 'dist/*', candidate module location: 'dist/src/index.js'.
Loading module as file / folder, candidate module location '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/index.js', target file type 'JavaScript'.
File name '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/index.js' has a '.js' extension - stripping it.
File '/Users/gozala/Projects/node_modules/bar.js' does not exist.
File '/Users/gozala/Projects/node_modules/bar.jsx' does not exist.
Directory '/Users/gozala/node_modules' does not exist, skipping all lookups in it.
Directory '/Users/node_modules' does not exist, skipping all lookups in it.
Directory '/node_modules' does not exist, skipping all lookups in it.
======== Module name 'bar' was not resolved. ========
src/main.ts:1:17 - error TS2307: Cannot find module 'bar' or its corresponding type declarations.

1 import Bar from "bar"
                  ~~~~~


Found 1 error.
```

It appers that if you create `node_modules/bar/src` directory typescirpt resolves module as expected, however current behavior still appears unexpected.


[typesVersions]:https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html#version-selection-with-typesversions
