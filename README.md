# React-Draggable

[![TravisCI Build Status](https://travis-ci.org/mzabriskie/react-draggable.svg?branch=master)](https://travis-ci.org/mzabriskie/react-draggable)
[![Appveyor Build Status](https://ci.appveyor.com/api/projects/status/32r7s2skrgm9ubva?svg=true)](https://ci.appveyor.com/project/mzabriskie/react-draggable)
[![npm downloads](https://img.shields.io/npm/dt/react-draggable.svg?maxAge=2592000)](http://npmjs.com/package/react-draggable)
[![gzip size](http://img.badgesize.io/https://npmcdn.com/react-draggable/dist/react-draggable.min.js?compression=gzip)]()
[![version](https://img.shields.io/npm/v/react-draggable.svg)]()

A simple component for making elements draggable.

```js
<Draggable>
  <div>I can now be moved around!</div>
</Draggable>
```

- [Demo](http://mzabriskie.github.io/react-draggable/example/)
- [Changelog](CHANGELOG.md)

------

#### Technical Documentation

- [Installing](#installing)
- [Exports](#exports)
- [Draggable](#draggable)
- [Draggable Usage](#draggable-usage)
- [Draggable API](#draggable-api)
- [Controlled vs. Uncontrolled](#controlled-vs-uncontrolled)
- [DraggableCore](#draggablecore)
- [DraggableCore API](#draggablecore-api)



### Installing

```bash
$ npm install react-draggable
```

If you aren't using browserify/webpack, a
[UMD version of react-draggable](dist/react-draggable.js) is available. It is updated per-release only.
This bundle is also what is loaded when installing from npm. It expects external `React` and `ReactDOM`.

If you want a UMD version of the latest `master` revision, you can generate it yourself from master by cloning this
repository and running `$ make`. This will create umd dist files in the `dist/` folder.

### Exports

The default export is `<Draggable>`. At the `.DraggableCore` property is [`<DraggableCore>`](#draggablecore).
Here's how to use it:

```js
// ES6
import Draggable from 'react-draggable'; // The default
import {DraggableCore} from 'react-draggable'; // <DraggableCore>
import Draggable, {DraggableCore} from 'react-draggable'; // Both at the same time

// CommonJS
let Draggable = require('react-draggable');
let DraggableCore = Draggable.DraggableCore;
```

## `<Draggable>`

A `<Draggable>` element wraps an existing element and extends it with new event handlers and styles.
It does not create a wrapper element in the DOM.

Draggable items are moved using CSS Transforms. This allows items to be dragged regardless of their current
positioning (relative, absolute, or static). Elements can also be moved between drags without incident.

If the item you are dragging already has a CSS Transform applied, it will be overwritten by `<Draggable>`. Use
an intermediate wrapper (`<Draggable><span>...</span></Draggable>`) in this case.

### Draggable Usage

View the [Demo](http://mzabriskie.github.io/react-draggable/example/) and its
[source](/example/index.html) for more.

```js
import React from 'react';
import ReactDOM from 'react-dom';
import Draggable from 'react-draggable';

class App extends React.Component {

  eventLogger = (e: MouseEvent, data: Object) => {
    console.log('Event: ', e);
    console.log('Data: ', data);
  };

  render() {
    return (
      <Draggable
        axis="x"
        handle=".handle"
        defaultPosition={{x: 0, y: 0}}
        position={null}
        grid={[25, 25]}
        onStart={this.handleStart}
        onDrag={this.handleDrag}
        onStop={this.handleStop}>
        <div>
          <div className="handle">Drag from here</div>
          <div>This readme is really dragging on...</div>
        </div>
      </Draggable>
    );
  }
}

ReactDOM.render(<App/>, document.body);
```

### Draggable API

The `<Draggable/>` component transparently adds draggability to its children.

**Note**: Only a single child is allowed or an Error will be thrown.

For the `<Draggable/>` component to correctly attach itself to its child, the child element must provide support
for the following props:
- `style` is used to give the transform css to the child.
- `className` is used to apply the proper classes to the object being dragged.
- `onMouseDown`, `onMouseUp`, `onTouchStart`, and `onTouchEnd`  are used to keep track of dragging state.

React.DOM elements support the above properties by default, so you may use those elements as children without
any changes. If you wish to use a React component you created, you'll need to be sure to
[transfer prop](https://facebook.github.io/react/docs/transferring-props.html).

#### `<Draggable>` Props:

```js
//
// Types:
//
type DraggableEventHandler = (e: Event, data: DraggableData) => void | false;
type DraggableData = {
  node: HTMLElement,
  // lastX + deltaX === x
  x: number, y: number,
  deltaX: number, deltaY: number,
  lastX: number, lastY: number
};

//
// Props:
//
{
// If set to `true`, will allow dragging on non left-button clicks.
allowAnyClick: boolean,

// Determines which axis the draggable can move. This only affects
// flushing to the DOM. Callbacks will still include all values.
// Accepted values:
// - `both` allows movement horizontally and vertically (default).
// - `x` limits movement to horizontal axis.
// - `y` limits movement to vertical axis.
// - 'none' stops all movement.
axis: string,

// Specifies movement boundaries. Accepted values:
// - `parent` restricts movement within the node's offsetParent
//    (nearest node with position relative or absolute), or
// - a selector, restricts movement within the targeted node
// - An object with `left, top, right, and bottom` properties.
//   These indicate how far in each direction the draggable
//   can be moved.
bounds: {left: number, top: number, right: number, bottom: number} | string,

// Specifies a selector to be used to prevent drag initialization.
// Example: '.body'
cancel: string,

// Class names for draggable UI.
// Default to 'react-draggable', 'react-draggable-dragging', and 'react-draggable-dragged'
defaultClassName: string,
defaultClassNameDragging: string,
defaultClassNameDragged: string,

// Specifies the `x` and `y` that the dragged item should start at.
// This is generally not necessary to use (you can use absolute or relative
// positioning of the child directly), but can be helpful for uniformity in
// your callbacks and with css transforms.
defaultPosition: {x: number, y: number},

// If true, will not call any drag handlers.
disabled: boolean,

// Specifies the x and y that dragging should snap to.
grid: [number, number],

// Specifies a selector to be used as the handle that initiates drag.
// Example: '.handle'
handle: string,

// If desired, you can provide your own offsetParent for drag calculations.
// By default, we use the Draggable's offsetParent. This can be useful for elements
// with odd display types or floats.
offsetParent: HTMLElement,

// Called whenever the user mouses down. Called regardless of handle or
// disabled status.
onMouseDown: (e: MouseEvent) => void,

// Called when dragging starts. If `false` is returned any handler,
// the action will cancel.
onStart: DraggableEventHandler,

// Called while dragging.
onDrag: DraggableEventHandler,

// Called when dragging stops.
onStop: DraggableEventHandler,

// Much like React form elements, if this property is present, the item
// becomes 'controlled' and is not responsive to user input. Use `position`
// if you need to have direct control of the element.
position: {x: number, y: number}
}
```


Note that sending `className`, `style`, or `transform` as properties will error - set them on the child element
directly.


## Controlled vs. Uncontrolled

`<Draggable>` is a 'batteries-included' component that manages its own state. If you want to completely
control the lifecycle of the component, use `<DraggableCore>`.

For some users, they may want the nice state management that `<Draggable>` provides, but occasionally want
to programmatically reposition their components. `<Draggable>` allows this customization via a system that
is similar to how React handles form components.

If the prop `position: {x: number, y: number}` is defined, the `<Draggable>` will ignore its internal state and use
the provided position instead. Alternatively, you can seed the position using `defaultPosition`. Technically, since
`<Draggable>` works only on position deltas, you could also seed the initial position using CSS `top/left`.

We make one modification to the React philosophy here - we still allow dragging while a component is controlled.
We then expect you to use at least an `onDrag` or `onStop` handler to synchronize state.

To disable dragging while controlled, send the prop `disabled={true}` - at this point the `<Draggable>` will operate
like a completely static component.

## `<DraggableCore>`

For users that require absolute control, a `<DraggableCore>` element is available. This is useful as an abstraction
over touch and mouse events, but with full control. `<DraggableCore>` has no internal state.

See [React-Resizable](https://github.com/STRML/react-resizable) and
[React-Grid-Layout](https://github.com/STRML/react-grid-layout) for some usage examples.

`<DraggableCore>` is a useful building block for other libraries that simply want to abstract browser-specific
quirks and receive callbacks when a user attempts to move an element. It does not set styles or transforms
on itself and thus must have callbacks attached to be useful.

### DraggableCore API

`<DraggableCore>` takes a limited subset of options:

```js
{
  allowAnyClick: boolean,
  cancel: string,
  disabled: boolean,
  enableUserSelectHack: boolean,
  offsetParent: HTMLElement,
  grid: [number, number],
  handle: string,
  onStart: DraggableEventHandler,
  onDrag: DraggableEventHandler,
  onStop: DraggableEventHandler,
  onMouseDown: (e: MouseEvent) => void
}
```

Note that there is no start position. `<DraggableCore>` simply calls `drag` handlers with the below parameters,
indicating its position (as inferred from the underlying MouseEvent) and deltas. It is up to the parent
to set actual positions on `<DraggableCore>`.

Drag callbacks (`onStart`, `onDrag`, `onStop`) are called with the [same arguments as `<Draggable>`](#draggable-api).

----

### Contributing

- Fork the project
- Run the project in development mode: `$ npm run dev`
- Make changes.
- Add appropriate tests
- `$ npm test`
- If tests don't pass, make them pass.
- Update README with appropriate docs.
- Commit and PR

### Release checklist

- Update CHANGELOG
- `make release-patch`, `make release-minor`, or `make-release-major`
- `make publish`

### License

MIT
