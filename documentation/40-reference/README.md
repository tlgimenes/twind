# Reference

## Lang

- grouping variants
  - `hover:(underline font-bold)` -> `hover:underline hover:font-bold`
- grouping utilities
  - `text-(sm green-500)` -> `text-sm text-green-500`
- grouping important
  - `!(text-(sm green-500))` -> `!text-sm !text-green-500`
- inline apply: styles are generated in order they are declared
  - `@(underline font-bold)` -> `@(underline,font-bold)`
  - `Link@(underline font-bold)` -> `Link#12345`
- inline shortcut: style are generated as defined by twind — same as if they where used alone
  - `~(underline font-bold)` -> `~(underline,font-bold)`
  - `Link~(underline font-bold)` -> `Link#abcdef`
- comments
  - multi line: `/* .. */`
  - single line: `//` — should **not** be used directly in a class attribute, use `cx` to _clear_ the class names before

## CSS (strings and objects)

- `&`: for nested selectors ([CSS Nesting Module](https://tabatkins.github.io/specs/css-nesting/))
- `label`: for a more readable class names [Emotion › Labels](https://emotion.sh/docs/labels)
- `theme(...)`: access theme values using dot notation; can be used in arbitrary values as well ([Tailwind CSS › Functions & Directives › theme()](https://tailwindcss.com/docs/functions-and-directives#theme))
- `@layer ...`: tell Twind which _bucket_ a set of custom styles belong to ([Tailwind CSS › Functions & Directives › layer](https://tailwindcss.com/docs/functions-and-directives#layer) & [Cascade Layers (CSS @layer) spec](https://www.bram.us/2021/09/15/the-future-of-css-cascade-layers-css-at-layer/))
  - The following layer exist in the given order: `defaults`, `base`, `components`, `shortcuts`, `utilities`, `overrides`
- `@apply`: inline any existing utility classes ([Tailwind CSS › Functions & Directives › apply](https://tailwindcss.com/docs/functions-and-directives#apply))
- `@media screen(...)`: create media queries that reference breakpoints by name instead of duplicating their values ([Tailwind CSS › Functions & Directives › screen()](https://tailwindcss.com/docs/functions-and-directives#screen))

## API

The following functions are all exports from `twind`.

### _Shim Mode_

Observe class attributes to inject styles

- `setup(config, sheet?, target?): Twind`: configures the global `tw` instance, observes all class attributes and inject there styles into the DOM; returns a twind instance
- `tw`: the global twind instance updated by each `setup` call

### _Library Mode_

use `tw` or `tx` to inject styles

- `twind(config, sheet?): Twind`: creates a custom twind instance (`tw`)

  Recommended custom twind pattern:

  ```js
  import {
    twind,
    cssom,
    virtual,
    tx as tx$,
    injectGlobal as injectGlobal$,
    keyframes as keyframes$,
  } from 'twind'

  import config from './twind.config'

  export const tw = /* @__PURE__ */ twind(
    config,
    // IS_SSR: `typeof document == 'undefined'` or `import.meta.env.SSR` (vite)
    // IS_PROD: `proces.env.NODE_ENV == 'production'` or `import.meta.env.PROD` (vite)
    IS_SSR ? virtual() : IS_PROD ? cssom() : dom(),
  )

  export const tx = /* @__PURE__ */ tx$.bind(tw)
  export const injectGlobal = /* @__PURE__ */ injectGlobal$.bind(tw)
  export const keyframes = /* @__PURE__ */ keyframes$.bind(tw)
  ```

- `observe(tw, target?)`: observes all class attributes and injects the styles into the DOM

### Twind instance — `tw`

- global twind instance: `import { tw } from 'twind'`
- `tw(className)`: injects a className string into the sheet and return the resulting class names
- `tw.theme(section?, key?, defaultValue?)`: access the current theme
  - `tw.theme()`: returns the whole thene
  - `tw.theme(section)`: returns the whole section
  - `tw.theme(dottedKey, defaultValue?)`: returns the current value
  - `tw.theme(section?, key?, defaultValue?)`: returns the theme value
- `tw.target`: the sheet target of this instance (`string[]`, `HTMLStyleSheet`, `CSSStyleSheet`)
- `tw.clear()`: clears all CSS rules from the sheet
- `tw.destroy()`: remove the sheet from the document

### Utilities

- `defineConfig(config)`: define a configuration object for `setup` or `twind`
- `tx(...args)`: creates a class name from the given arguments and injects the styles (like `tw(cx(...args))`)

  - `tx.bind(tw)`: binds the `tx` function to a custom twind instance; returns a new `tx` function

  ```js
  import { tx } from 'twind'

  const className = tx`
    color: red;
    font-size: 12px;
  `
  ```

- `injectGlobal(...args)`: injects the given styles into the base layer

  - `injectGlobal.bind(tw)`: binds the `injectGlobal` function to a custom twind instance; returns a new `injectGlobal` function

  ```js
  import { injectGlobal } from 'twind'

  injectGlobal`
    @font-face {
      font-family: "Operator Mono";
      src: url("../fonts/Operator-Mono.ttf");
    }
  
    body {
      margin: 0;
    }
  `
  ```

- `keyframes(...args)`: lazily injects the keyframes into the sheet and return a unique name

  - `keyframes.Name(tw)`: lazily injects the named keyframes into the sheet and return a unique name
  - `keyframes.bind(tw)`: binds the `keyframes` function to a custom twind instance; returns a new `keyframes` function

  ```js
  import { keyframes, css, tx } from 'twind'

  const fadeIn = keyframes`
    0% {
      opacity: 0;
    }
    100% {
      opacity: 1;
    }
  `

  // if in _Shim Mode_
  const fadeInClass = css`
    animation: 1s ${fadeIn} ease-out;
  `

  // if in _Library Mode_
  const fadeInClass = tx`
    animation: 1s ${fadeIn} ease-out;
  `
  ```

### Helper functions

These generate class names but do **not** inject styles.

These can be used to generate class names that are then

a) set the class attribute on an element (_Shim Mode_)<br>
b) used with `tw` to inject styles and return a class name (_Library Mode_)

- `cx(...args)`: creates a class name from the given arguments; no styles injected
- `css(...args)`: creates a class name from the given arguments; no styles injected
- `shortcut(...args)`: creates a class name from the given arguments; order of styles determined by twind; no styles injected
  - `shortcut.Button(...args): creates a named class name from the given arguments; order of styles determined by twind; no styles injected
  - `~(...)` or `Button~(...)`: within a token
- `apply(...args)`: creates a class name from the given arguments; order of styles determined by order in args; no styles injected
  - `apply.Button(...args): creates a named class name from the given arguments; order of styles determined by order in args; no styles injected
  - `@(...)` or `Button@(...)`: within a token
  - `@apply ...` or `{ '@apply': '...' }`: within CSS string or object
- `style(options)`: creates a stitches like helper; returns a `style` function
  - `style(props)`: creates a class name from the given props; no styles injected

### Mostly server side

Used to update an html string with styles.

- `inline(html, tw? | {tw, minfiy}?)`: updates all class attributes in html string and inject there styles into the head as style element; returns the updated html string
- `extract(html, tw?)`: updates all class attributes from html string; returns the updated html string and the CSS
- `consume(html, tw?)`: updates all class attributes from html string; returns the updated html string and injects all styles into `tw`

### Sheets

- `virtual()`: collect styles into an array
- `cssom()`: uses a fast DOM sheet — bad for debugging
- `dom()`: uses a slow DOM sheet — great for debugging
- `stringify(target)`: returns the CSS string of a sheet target

## Config

- hash all shortcuts and apply

  ```js
  setup({
    hash(className, defaultHash) {
      if (/^[~@]\(/.test(className)) {
        // a shortcut like `~(...)`
        // an apply like `@(...)`
        return defaultHash(className)
      }

      return className
    },
  })
  ```

## Browser Support

In general, Twind is designed for and tested on the latest stable versions of Chrome, Firefox, Edge, and Safari. It does not support any version of IE, including IE 11.

For automatic vendor prefixing include the [@twind/preset-autoprefix](https://www.npmjs.com/package/@twind/preset-autoprefix) preset.

For more details see [Tailwind CSS › Browser Support](https://tailwindcss.com/docs/browser-support).

The following JS APIs may need polyfills:

- [Array.flatMap](https://caniuse.com/mdn-javascript_builtins_array_flatmap)
  - Edge<79, Firefox<62, Chrome<69, Safari<12, Opera<56
  - [polyfill](https://www.npmjs.com/package/array-flat-polyfill)
- [Math.imul](https://caniuse.com/mdn-javascript_builtins_math_imul)
  - Firefox<20, Chrome<28, Safari<7, Opera<16
  - [polyfill](https://www.npmjs.com/package/math.imul)