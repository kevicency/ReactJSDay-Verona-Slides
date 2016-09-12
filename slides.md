# CSS is dead! <br/> Long live
<img id="css-modules-logo" data-src="images/css-modules-logo.png" />

<p id="author">
  by Kevin Mees /
  <img src="./images/github-logo.png" />kmees
</p>



<!-- .slide: data-transition="fade" -->
## Let's talk about <br/> the current state of CSS


<!-- .slide: data-transition="fade" data-background="images/globals-everywhere.jpg" data-background-size="contain" -->
## Let's talk about <br/> the current state of CSS


## Global state is bad!
* No isolation <br/>
  &rarr; 3rd party libs can break your stuff or vice versa
* Dependency on inclusion / load order <br/>
  &rarr; Nasty bugs when dynamically loading additional CSS


## ~~Solutions~~ Workarounds
* Architecture Guidelines (BEM, SMACSS, OOCSS, ...)
* Preprocessors (SASS, LESS, Stylus, ...)

  <br/> &rarr; They do not solve the problem of global state but they make it more manageable.

Note:
* Architecture Guidelines minimize the risk of creating global conflicts in your codebase
* Preprocessors help implementing those guidelines



<!-- .slide: data-background="images/mind-blown.gif" data-background-size="contain" -->
### JS had the same problem(s) few years ago!


<!-- .slide: data-transition="fade" -->
## Let's talk about <br/> the current state of JS


<!-- .slide: data-transition="fade" data-background="images/modules-everywhere.jpg" data-background-size="contain" -->
## Let's talk about <br/> the current state of JS
  


## JS Modules in Action
```sh
npm i --save react
```
```js
/* MyComponent.js */
import React from 'react'

export default class MyComponent extends React.Component {
  render() {
    return <div>My Component</div>
  }
}
```


### Can we apply this to CSS?
```sh
npm i --save bootstrap
```
```js
// Button.js
import style from 'bootstrap/dist/bootstrap.css'

export default const Button = ({ children }) => (
  <button className={${style['btn'] + ' ' + style['btn-default']}>
    { children }
  </button>
)
```


### What does importing Stylesheet do?
```js
import style from 'bootstrap/dist/bootstrap.css'

assert.equal(style, {
  'btn': '_23_aKvs-b8bW2Vg3fwHozO',
  'btn-default': '_13LGdX8RMStbBE9w-t0gZ1',
  'btn-primary': '_53_ade-Zm9vYmFyYmF6oPx'
  // ...
})
```
* Class names in the stylesheet are **local**
* The style object is a map of <br/>
  **local** (`.btn`) &rarr;  **global** (`._23_aKvs-b8bW2Vg3fwHozO`)
* **global** class names are random hashes &rarr; no collisions
* The stylesheet with the **global** class names gets added to the document



<!-- .slide: data-transition="fade" -->
## Can we do it?


<!-- .slide: data-background="images/we-can-do-it.jpg" data-background-size="contain" data-transition="fade" -->
## Can we do it?


### Enter webpack & css-loader
```sh
npm i -D webpack css-loader
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
```js
import Button from './Button.js'
React.render(<Button>Foo</Button>, body)
```
```html
<button class="_23_aKvs-b8bW2Vg3fwHozO _13LGdX8RMStbBE9w-t0gZ1">
    Submit
</button>
```


<!-- .slide: data-background="images/giphy-confused.gif" data-background-size="contain" -->
### Hashes during development?


### Development
* Use **`localIdentName`** query paramter to customize the generated ident.
* For development:

```
css-loader?module&localIdentName=[path][name]__[local] 
```
```html
<button class="node_modules-bootstrap-dist-bootstrap__btn
               node_modules-bootstrap-dist-bootstrap__btn-default">
  Foo
</button>
```


<!-- .slide: data-background="images/confused.gif" data-background-size="contain" -->
### Wait... <br/> do I need to bundle my unit tests now?!


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



<!-- .slide: data-background="images/candy.jpg" data-background-size="contain" -->
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
