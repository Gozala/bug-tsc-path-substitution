# Problem

Typescript's [`typesVersions`](typesVersions) allows package to map generated `.d.ts` files produces by `tsc --emitDeclartions` so that users of the package will get all the typings out of the box.

However it seems that path substitution does not seems to fail when module is importing a directory with `index` file in it. This repo illustartes that problem:

Module `src/index.ts` imports `bar/src` module which seems to do following:

1. typesVersion substitution is applied to 'src', which matched pattern '*'
   > Trying substitution 'dist/*', candidate module location: 'dist/src'.
2. Which triggers following lookups:
  - `node_modules/bar/dist/src.ts`
  - `node_modules/bar/dist/src.tsx`
  - `node_modules/bar/dist/src.d.ts`
3. After neither found instead of looking up at `node_modules/bar/dist/src/index.(ts,tsx,d.ts)` it seems to look at 'package.json' 'main' field and do another path substitution:
   > 'package.json' has 'main' field 'src/index.js' that references 'node_modules/bar/dist/src/src/index.js'.
4. Which fails because it ends up with `dist/src/src/index` instead of `dist/src/index`.
5. This is especially problematic because as `node_modules/foo/` illustrates TS generates typedefs that add turn import for `bar` into `bar/src`. 

Complete module resolution trace below:
```
npx typescript@next --noEmit --traceResolution
npx: installed 1 in 1.091s
======== Resolving module 'bar/src' from '/Users/gozala/Projects/bug-tsc-double-substitution/src/main.ts'. ========
Explicitly specified module resolution kind: 'NodeJs'.
Loading module 'bar/src' from 'node_modules' folder, target file type 'TypeScript'.
Directory '/Users/gozala/Projects/bug-tsc-double-substitution/src/node_modules' does not exist, skipping all lookups in it.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/src/package.json' does not exist.
Found 'package.json' at '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/package.json'.
'package.json' has a 'typesVersions' field with version-specific path mappings.
'package.json' has a 'typesVersions' entry '*' that matches compiler version '4.1.0-dev.20201027', looking for a pattern to match module name 'src'.
Module name 'src', matched pattern '*'.
Trying substitution 'dist/*', candidate module location: 'dist/src'.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src.ts' does not exist.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src.tsx' does not exist.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src.d.ts' does not exist.
'package.json' does not have a 'typings' field.
'package.json' does not have a 'types' field.
'package.json' has 'main' field 'src/index.js' that references '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/src/index.js'.
'package.json' has a 'typesVersions' entry '*' that matches compiler version '4.1.0-dev.20201027', looking for a pattern to match module name 'src/index.js'.
Module name 'src/index.js', matched pattern '*'.
Trying substitution 'dist/*', candidate module location: 'dist/src/index.js'.
Loading module as file / folder, candidate module location '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/dist/src/index.js', target file type 'TypeScript'.
File name '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/dist/src/index.js' has a '.js' extension - stripping it.
Directory '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/@types' does not exist, skipping all lookups in it.
Directory '/Users/gozala/Projects/node_modules/@types' does not exist, skipping all lookups in it.
Directory '/Users/gozala/node_modules' does not exist, skipping all lookups in it.
Directory '/Users/node_modules' does not exist, skipping all lookups in it.
Directory '/node_modules' does not exist, skipping all lookups in it.
Loading module 'bar/src' from 'node_modules' folder, target file type 'JavaScript'.
Directory '/Users/gozala/Projects/bug-tsc-double-substitution/src/node_modules' does not exist, skipping all lookups in it.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/src/package.json' does not exist.
Found 'package.json' at '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/package.json'.
'package.json' has a 'typesVersions' field with version-specific path mappings.
'package.json' has a 'typesVersions' entry '*' that matches compiler version '4.1.0-dev.20201027', looking for a pattern to match module name 'src'.
Module name 'src', matched pattern '*'.
Trying substitution 'dist/*', candidate module location: 'dist/src'.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src.js' does not exist.
File '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src.jsx' does not exist.
'package.json' has 'main' field 'src/index.js' that references '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/src/index.js'.
'package.json' has a 'typesVersions' entry '*' that matches compiler version '4.1.0-dev.20201027', looking for a pattern to match module name 'src/index.js'.
Module name 'src/index.js', matched pattern '*'.
Trying substitution 'dist/*', candidate module location: 'dist/src/index.js'.
Loading module as file / folder, candidate module location '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/dist/src/index.js', target file type 'JavaScript'.
File name '/Users/gozala/Projects/bug-tsc-double-substitution/node_modules/bar/dist/src/dist/src/index.js' has a '.js' extension - stripping it.
Directory '/Users/gozala/node_modules' does not exist, skipping all lookups in it.
Directory '/Users/node_modules' does not exist, skipping all lookups in it.
Directory '/node_modules' does not exist, skipping all lookups in it.
======== Module name 'bar/src' was not resolved. ========
src/main.ts:1:17 - error TS2307: Cannot find module 'bar/src' or its corresponding type declarations.

1 import Bar from "bar/src"
                  ~~~~~~~~~


Found 1 error.
```

It appers that if you create changing `typesVersions` from `"*": "dist/*"` to
`"*": ["dist/*", "dist/*/index"]` works around this problem, but current
behavior still seems incorrect.


[typesVersions]:https://www.typescriptlang.org/docs/handbook/declaration-files/publishing.html#version-selection-with-typesversions
