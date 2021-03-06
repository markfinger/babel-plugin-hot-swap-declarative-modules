# babel-plugin-hot-swap-declarative-modules

A babel plugin that enables hot swaps for declarative modules by inserting
`module.hot.accept()` at the top of the file.

Unlike `react-hot-loader` or `react-transform-hmr`, this plugin does not aim to
transition state between hot swaps. Instead, it simply aims to remove boilerplate.
Works best with a code-base that consolidates state into a single location.


## Installation

```
npm install --save babel-plugin-hot-swap-declarative-modules
```

Add `"hot-swap-declarative-modules"` to the plugins in your `.babelrc`:

```
{
  "plugins": [
    "hot-swap-declarative-modules"
  ]
}
```


## Caveats & Explanation

The structural scan is quite naive and scan does not traverse beyond the
most superficial level, so false positives are not the most difficult thing
to achieve.

Currently, the scan involves a simple iteration over the root of the module
and bails if it encounters anything but es2015 import/export declarations,
variable declarations (including var, let and const), class declarations or
function declarations.

If the iteration completes without encountering a non-declarative item, it
injects a `module.hot.accept` call. The `accept` call is placed at the top
of the file so that synchronous failures during evaluation of dependencies
does not prevent a module from accepting further hot swaps.

An example of the language features that are inferred as declarative is

```js
import a from './a';
export {b} from './b';

const c = 'c';

export const d = c + 'd';

export default function() {
  // ...
}

export class e extends a {
  // ...
}
```

An example of the language features that are inferred as non-declarative is

```
a();

if (b > 10) {
  // ...
}

const c = {};
c.d = 'd';
```

In short: conditionals, global variables, property assignments and function
calls should be avoided at the root level.


## Exceptions

Mindful of assisting with debugging, calls to the `console` object's properties
are assumed to be declarative.