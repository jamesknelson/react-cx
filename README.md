# react-cx

A utility to style React components with ~~style~~ css.

Inspired by [jareware/css-ns](https://github.com/jareware/css-ns). The hard work is done by [jedwatson/classnames](https://github.com/JedWatson/classnames).

Install with npm:

```sh
# Adds react-cx to node_modules and package.json
npm install react-cx --save
```

With react-cx, you can add styles from CSS modules by specifying a `cx` prop on you React elements:

```jsx
// Import a CSS file using CSS Modules
import styles from './Theme.less'

// Wrap `React` to add a `cx` prop that gets class names from the `styles`
import getReactWithCX from './react-cx'
const React = getReactWithCX(styles)

export function Arrow({ active, className, color, headless, noMobile, style, length=1 }) {
  return (
    <div
      // The `cx` props arguments are passed to the `classnames` package
      cx={['Arrow', { active, headless, 'no-mobile': noMobile }, color, 'length-'+length]}

      // If you pass a `classname`, it is appended as-is to any class names
      // from the `cx` prop.
      className={className}
     />
  )
}
```

*This arrow component is used in my [React lifecycle simulators](https://reactarmory.com/guides/lifecycle-simulators).*

## How does it work?

In React, JSX calls are translated to calls to `React.createElement()`. Thus, by wrapping `React.createElement()` with our own function, we can add support for new props without ever touching React's internals.

If you understand how `React.createElement()` works, the easiest way to grok this is to just view the source below. If you're not yet familiar with `React.createElement()`, head over to [React Armory](https://reactarmory.com/guides/learn-react-by-itself/react-basics) for a simple explanation with live examples and exercises.

```js
import UnprefixedReact from 'react'
import classnames from 'classnames/bind'

export default function cx(styles, prop='cx') {
  const bound = classnames.bind(styles)

  function getProps(props) {
    if (!props) return props

    if (props.cx) {
      const result = Object.assign({}, props)
      delete result.cx
      const args = Array.isArray(props.cx) ? props.cx : [props.cx]
      result.className = bound(...args)
      if (props.className) result.className += ' '+props.className
      return result
    }
    return props
  }
  function createElement(type, props, ...children) {
    return UnprefixedReact.createElement(type, getProps(props), ...children)
  }
  function cloneElement(element, props, ...children) {
    return UnprefixedReact.cloneElement(element, getProps(props), ...children)
  }
  return Object.assign({}, UnprefixedReact, {
    createElement,
    cloneElement
  })
}
```

## License

[/LICENSE] Copyright &copy; 2017 James K Nelson