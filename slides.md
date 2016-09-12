# CSS is dead! <br/> Long live
<img id="css-modules-logo" data-src="images/css-modules-logo.png" />

<p id="author">
  by Kevin Mees /
  <img src="./images/github-logo.png" />kmees
</p>



<!-- .slide: data-transition="fade" -->
## The biggest problem with CSS?


<!-- .slide: data-transition="fade" data-background="images/globals-everywhere.jpg" data-background-size="cover" -->
## The biggest problem with CSS


## Global state is bad!
* No isolation <br/>
  &rarr; 3rd party libs can break your stuff or vice versa
* Dependency on inclusion / load order <br/>
  &rarr; Nasty bugs when dynamically loading additional CSS
* and many more...


## ~~Solutions~~ Workarounds
* Architecture Guidelines (BEM, SMACSS, OOCSS, ...)
* Preprocessors (SASS, LESS, Stylus, ...)

  <br/> &rarr; They do not solve the problem of global state but they make it more manageable. 

Note:
* Architecture Guidelines minimize the risk of creating global conflicts in your codebase
* Preprocessors help implementing those guidelines



<!-- .slide: data-background="images/mind-blown.gif" data-background-size="cover" -->
## JS had the same problem(s) <br /> just a few years ago!


<!-- .slide: data-transition="fade" -->
## What killed the globals in JS?


<!-- .slide: data-transition="fade" data-background="images/modules-everywhere.jpg" data-background-size="cover" -->
## Modules did!

Note:
* The adaption of npm for client side development with the help of webpack
* Can we use the same approch to get rid of the globals in CSS?
  


## JS Modules in Action
```sh
npm i --save react react-dom
```
```js
import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <div>Hello World!</div>,
  document.body
);
```

Note:
Just a quick reminder...


### Applying the idea of JS Modules to CSS
* Class names in a stylesheet are **local** by default.

  ```sh
  npm i --save bootstrap
  ```
  <!-- .element: class="fragment" data-fragment-index="1" -->
  ```js
  import style from 'bootstrap/dist/bootstrap.css'

  style === {
    'btn': '_23_aKvs-b8bW2Vg3fwHozO',
    'btn-default': '_13LGdX8RMStbBE9w-t0gZ1',
    'btn-primary': '_53_ade-Zm9vYmFyYmF6oPx',
    // ...
  };
  ``` 
  <!-- .element: class="fragment" data-fragment-index="3" -->
<li class="fragment" data-fragment-index="3">
  The style object is a map of <br/> **local** (`.btn`) &rarr;  **global** (`._23_aKvs-b8bW2Vg3fwHozO`)
</li>
<li class="fragment" data-fragment-index="4">
  **global** class names are random hashes &rarr; no collisions
</li>
<li class="fragment" data-fragment-index="5">
  The final *generated* stylesheet has all **local** class names replaced with the **global** class names.
</li>



### Setting up Webpack
  ```sh
  npm i -D css-loader
  ```
  ```js
  // webpack.config.js
  export const config = {
      // ...
      loaders: [
        { test: /\.css$/, loader: "css-loader?module" }
      ]
  }
  ```

* **`?module`** enables CSS Modules support


### Using CSS Modules
```js
// Button.js
import style from 'bootstrap/dist/bootstrap.css'
/** {
  'btn': '_23_aKvs-b8bW2Vg3fwHozO',
  'btn-default': '_13LGdX8RMStbBE9w-t0gZ1'
}; */

export default const Button = ({ children }) => {
  const className = style['btn'] + ' ' + style['btn-default'];

  return (
    <button className={className}>
      { children }
    </button>
  )
}
```
```js
import Button from './Button.js'
React.render(<Button>Submit</Button>, body)
```
<!-- .element: class="fragment" data-fragment-index="1" -->
```html
<button class="_23_aKvs-b8bW2Vg3fwHozO _13LGdX8RMStbBE9w-t0gZ1">
    Submit
</button>
```
<!-- .element: class="fragment" data-fragment-index="2" -->

Note:
Import a stylesheet which gives us a reference to the styles


<!-- .slide: data-background="images/giphy-confused.gif" data-background-size="cover" -->
### This makes developing / debugging a hell!

Note:
You might be thinking: 'This is nice for production, but...'


### During Development
* Use **`localIdentName`** query paramter to customize the generated ident.
* Development example:

```
css-loader?module&localIdentName=[path][name]__[local] 
```
```html
<button class="node_modules-bootstrap-dist-bootstrap__btn
               node_modules-bootstrap-dist-bootstrap__btn-default">
  Submit
</button>
```


<!-- .slide: data-background="images/confused.gif" data-background-size="cover" -->
### Awesome, but wait... <br/> do I need to bundle my unit tests now?!


### Unit Testing
* Use `css-modules/css-modules-require-hook` to require css modules during runtime.

```js
/* test-setup.js */
const hook = require('css-modules-require-hook')

hook({
  generateScopedName: '[local]',
})
```

* **`generateScopedName`** behaves like **`localIdentName`**

```
/* button.test.js */
import Button from './Button.js'
import { shallow } from 'enzyme'

test('Button', t => {
  const button = shallow(<Button>Test</Button>)
  t.ok(button.hasClass('btn'))
  t.ok(button.hasClass('btn-default'))
})
```


### How do you include <br/> the global styles in your document?
* Use **`style-loader`** to add them as inline `<style />`s
* Use **`extract-text-webpack-plugin`** to extract them to a stylesheet
```
import ExtractTextPlugin from 'extract-text-webpack-plugin'
export const config = {
    // ...
    loaders: [
      { test: /\.css$/, loader: "style-loader!css-loader?module" }
    ],
    plugins: [
        new ExtractTextPlugin("styles.css")
    ]
}
```



<!-- .slide: data-background="images/candy.jpg" data-background-size="cover" -->
## Want some sugar on top?


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
* PostCSS for additional syntax (e.g. variables)
* CSSNext


### Better integration with React
```
import CSSModules from 'react-css-modules'
import style from 'bootstrap/dist/bootstrap.css'

const Button = ({ children }) => (
  <div styleName="btn btn-default"
       className="some-global-class">
    { children }
  </div>
)
export default CSSModules(Button, style)
```

* No more
  ```js
  className={`${style['btn']} ${style['btn-default']}`}
  ```
* `styleName` for **local** styles, `className` for **global** styles.
* Bonus: detects undefined local class names during runtime.



# Fin
Thank you for listening!

<p id="author">
  Slides @ <img src="./images/github-logo.png" />[kmees/Slides-ReactNext2016](https://github.com/kmees/ReactNext2016-Slides)
</p>
