## CSS is dead! <br/> &mdash; <br/> Long live CSS!
<small>*but in modules, please!*</small>

<p id="author">
  by <img alt="Experts Inside" src="./images/xi-logo_32.png" />Kevin Mees /
  <img alt="GitHub" src="./images/github-logo.png" />kmees
</p>



<!-- .slide: data-transition="fade" -->
## What makes working with CSS so hard?


# &nbsp;  
<!-- .slide: data-transition="fade" data-background="images/globals-everywhere.jpg" data-background-size="cover" -->


## Global state makes developers sad!


## ~~Solutions~~ Workarounds
* Architecture Guidelines (BEM, SMACSS, OOCSS, ...)
* Preprocessors (SASS, LESS, Stylus, ...)

  <br/> &rarr; They **do not solve** the problem of global state but they make it more **manageable**. 

Note:
* The funny thing is that JS had the same problems a few years ago



<!-- .slide: data-background="images/mind-blown.gif" data-background-size="cover" -->
## JS had the same problem(s) <br /> just a few years ago!

Note:
* But the problem is mostly solved by now


<!-- .slide: data-transition="fade" -->
## What killed the globals in JS?


<!-- .slide: data-transition="fade" data-background="images/modules-everywhere.jpg" data-background-size="cover" -->
## Modules did!

Note:
* Can we use the same approch to get rid of the globals in CSS?
  


### Applying the concept of JS Modules to CSS
```sh
npm i --save bootstrap
```

```js
import style from 'bootstrap/dist/bootstrap.css'
```

<ul>
  <li class="fragment" data-fragment-index="1">
    By importing a stylesheet as a module we get a **local** reference to it.
  </li>
  <li class="fragment" data-fragment-index="2">
    All class names in this stylesheet module are **local** by default. 
  </li>
  <li class="fragment" data-fragment-index="3">
    CSS only knows **global**, how can we safely get there? 
  </li>
</ul>


### What is a stylesheet module?
```js
import style from 'bootstrap/dist/bootstrap.css'
```

```js
style === {
  'btn': '_23_aKvs-b8bW2Vg3fwHozO',
  'btn-primary': '_13LGdX8RMStbBE9w-t0gZ1',
  'btn-default': '_53_ade-Zm9vYmFyYmF6oPx',
  // ...
};
``` 
<!-- .element: class="fragment" data-fragment-index="1" -->

<ul>
  <li class="fragment" data-fragment-index="1">
    The stylesheet module is a map of <br/> **local** (`btn`) &rarr;  **global** (`_23_aKvs-b8bW2Vg3fwHozO`)
  </li>
  <li class="fragment" data-fragment-index="2">
    **global** class name is a hash of the **local** class name <br />
    with the stylesheet added as salt <br />
    &rarr; **local** class names can be reused across stylesheets
  </li>
  <li class="fragment" data-fragment-index="3">
    The stylesheet that will be added to the document will be  *generated*, 
    with all **local** class names replaced by their corresponding **global** class name.
  </li>
</ul>


### Same **local** &rarr; different **global**
```css
/* dialog.css */
.btn { /* ... */ }

/* form.css */
.btn { /* ... */ }
```
```js
import dialogStyle from './dialog.css'
import formStyle from './form.css'

console.log(dialogStyle['btn'])
// => '_62_gOwm-q3lJt6g4FmtI4g',
console.log(formStyle['btn'])
// => '_47_bDog-e5bJ5Qg5fkBidO',

assert.notEqual(dialogStyle['btn'], formStyle['btn'])
```




## How can we actually use that?


### Setting up Webpack
  ```sh
  npm i -D css-loader
  ```
  ```js
  // webpack.config.js
  export const config = {
      // ...
      loaders: [
        { test: /\.css$/, loader: "css?module" }
      ]
  }
  ```

* **`?module`** enables CSS Modules support


### CSS Modules Example
```js
// Button.js
import style from 'bootstrap/dist/bootstrap.css'
/** {
  'btn': '_23_aKvs-b8bW2Vg3fwHozO',
  'btn-default': '_13LGdX8RMStbBE9w-t0gZ1'
}; */
```
<!-- .element: class="fragment" data-fragment-index="0" -->

```js
export default const Button = ({ children }) => {
  const className = style['btn'] + ' ' + style['btn-primary'];

  return (
    <button className={className}>
      { children }
    </button>
  )
}
```
<!-- .element: class="fragment" data-fragment-index="1" -->

```js
React.render(<Button>Submit</Button>, body)
```
<!-- .element: class="fragment" data-fragment-index="2" -->
```html
<button class="_23_aKvs-b8bW2Vg3fwHozO _13LGdX8RMStbBE9w-t0gZ1">
    Submit
</button>
```
<!-- .element: class="fragment" data-fragment-index="3" -->

Note:
* Using the **local** class name **`btn`** to get a reference to the **global** class name.


<!-- .slide: data-background="images/giphy-confused.gif" data-background-size="cover" -->
### Is there a human readable version of that?

Note:
You might be thinking: 'This is nice for production, but...'


### Development Setup
* Use **`localIdentName`** query paramter to customize the generated ident.

```js
{ loader: 'css?module&localIdentName=[path][name]__[local]' }
```
```html
<button class="node_modules-bootstrap-dist-bootstrap__btn
               node_modules-bootstrap-dist-bootstrap__btn-primary">
  Submit
</button>
```


<!-- .slide: data-background="images/confused.gif" data-background-size="cover" -->
### Awesome, but wait... <br/> do I need to bundle my unit tests now?!


### Unit Testing Setup
```
/* button.test.js */
import Button from './Button.js'
import { shallow } from 'enzyme'
import test from 'tape'

test('Button', t => {
  const button = shallow(<Button>Test</Button>)

  t.ok(button.hasClass('btn'))
  t.ok(button.hasClass('btn-primary'))
})
```
* This test will throw a runtime error because node doesn't
  know how to require CSS files


### Unit testing setup

* Use **`css-modules-require-hook`** to require 
  CSS files as modules in node during runtime.
* **`generateScopedName`** behaves like **`localIdentName`** for css-loader

```js
/* test-setup.js */
const hook = require('css-modules-require-hook')

hook({ generateScopedName: '[local]' })
```
* No need to update existing unit tests


### How do you include <br/> the global stylesheet in your document?
* Use **`style-loader`** to include it as inline `<style />`
```
export const config = {
    loaders: [
      { test: /\.css$/, loader: "style!css?module" }
    ]
}
```
* Use **`extract-text-webpack-plugin`** to write it to a file
```
import ExtractTextPlugin from 'extract-text-webpack-plugin'
export const config = {
    loaders: [
      { test: /\.css$/, 
        loader: ExtractTextPlugin.extract('style', 'css?module') }
    ],
    plugins: [
        new ExtractTextPlugin("styles.css")
    ]
}
```



<!-- .slide: data-background="images/candy.jpg" data-background-size="cover" -->
## Want some sugar on top?


### React integration via HoC
* Primitive implementation

```
import style from 'bootstrap/dist/bootstrap.css'

const Button = ({ className = '', children }) => {
  const btnClassName = style['btn'] + ' ' + style['btn-primary']

  return (
    <button className={btnClassName + ' ' + className}>
      { children }
    </button>
  )
}

export default Button
```

* Dealing with the `style` object directly is tedious
* A lot of string concatenations


### React integration via HoC
* Implementation with `react-css-modules`

```
import CSSModules from 'react-css-modules'
import style from 'bootstrap/dist/bootstrap.css'

const Button = ({ className, children }) => (
  <button className={className}
    styleName="btn btn-primary">
    { children }
  </button>
)

export default CSSModules(Button, style)
```

* `styleName` for **local** styles, `className` for **global** styles
* Bonus: detects undefined **local** class names during runtime <br />
  <img class="no-border" data-src="./images/css-modules-error.png">


### Composition
* Compose class names in the same file
  ```css
  /* button.css */
  .button {
    color: white;
    background: gray;
  }
  .button-primary {
    composes: button;
    background: blue;
  }
  ```
* or class names from other CSS modules
  ```css
  /* dialog.css */
  .dialog-button {
    composes: button from './button.css';
    background: green;
  }
  ```


### Works flawlessly with
* Preprocessors
  ```js
  { test: /\.css$/, loader: "css?module!sass" }
  ```
* PostCSS for additional syntax (e.g. variables)
  ```js
  { test: /\.css$/, loader: "css?module!postcss" }
  ```
* CSS Linters
* ...



## CSS Modules vs CSS in JS


### What is CSS in JS
* First proposed by @vjeux back in late 2014 
* Eliminate the global state in CSS by writing CSS in JS
* Render the CSS as inline styles
* A lot of implementations followed: Comparison by Michele Bertoli @ <img class="github-logo" src="./images/github-logo.png">[MicheleBertoli/css-in-js](https://github.com/MicheleBertoli/css-in-js)


### The problems with CSS in JS
* Bad tooling for writing CSS in JS
* Awkward to write when you like camel-case for CSS classes
* Inline styles don't support all CSS features and <br />
  generate **a lot** of bloat in your DOM
* No reuse of existing CSS without converting it first
* **The more sophisticated solutions add (too) much complexity to you project**



## Personal Experience with CSS Modules
* Wrote one open-source react component library for Office UI Fabric using CSS Modules
* 'Modulified' (parts of) one internal project which now uses a mix of <br />
  local and global css
* Used CSS Modules in two small Outlook Add-Ins implemented with React



# Fin
Thanks for listening!

<p id="author">
  Slides @ <img src="./images/github-logo.png" />[kmees/Slides-ReactNext2016](https://github.com/kmees/ReactNext2016-Slides)
</p>
