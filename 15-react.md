# React

Our react setup uses React, redux, Immutable.js and react-router.

## Pods

We organize our React components inside of directories called pods, a name borrowed
from Ember. Each pod contains at least one main file that exports a component and
can be referenced from other files using `import`;

The most basic pod is a javascript file that exports a stateless functional component:

```
// SomeComponent.js
export default function SomeComponent() {}
```

If necessary a pod can contain the following files:

### SomeComponent/index.js

This file exports a React component. It hides the implementation of `SomeComponent`
using requires or imports.

### SomeComponent/connector

The `connector` file exports a function that takes a component class as an argument
and returns another component class. This connector usually connects to the redux-state
using `import { connect } from 'react-redux';`

#### Testing

The tests should check that the correct state is passed down to the Component.

```
describe('SomeComponent/connector', () => {
  it('passes the current user to the component', () => {
    function TestComponent(props) {}
    const ConnectedComponent = connector(TestComponent);
    const renderer = TestUtils.createRenderer();
    renderer.render(<ConnectedComponent name="prop1" />);
    const output = renderer.getRenderOutput();
    expect(findWithType(output, ConnectedComponent).props).to.deep.equal({
      name: 'prop1',
      currentUser: 'current user',
    });
  });
});
```

### SomeComponent/component

A stateless component that takes an arbitrary props and context and renders some markup.
This component can delegate to other components or utilize private functions.

#### Testing

The tests should check that the correct markup gets generated given some props as
an argument.

```
describe('SomeComponent/component', () => {
  it('shows the user name', () => {
    const renderer = TestUtils.createRenderer();
    renderer.render(<Component user={...} />);
    const output = renderer.getRenderOutput();
    expect(findWithType(output, "h1").children).to.equal('Cool dude');
  });
});
```


### SomeComponent/controller

The controller file is a react component that can take advantage of the React lifecycle
functions. It can also define methods that are passed as props to other components
in the render method.

The render method in a controller should not render markup, but instead delegates
to a component that does. Different components can be rendered based on
the state and prop of the controller and the props that they received can
also change based on the state.

A controller should test the rendered component as well as all methods that are
part of its public interface.

#### Render tests

```
describe('SomeComponent/controller', () => {
  function render(props) {
    const renderer = TestUtils.createRenderer();
    renderer.render(<NotificationController {...props} />);
    return renderer;
  }

  describe('#render', () => {
    it('renders the EmptyComponent when no items', () => {
      const output = render({ items: [], isLoadingItems: true }).getRenderOutput();
      expect(findWithType(output, EmptyComponent).props).to.deep.equal({
        isLoading: true,
      });
    });
  });
});
```

#### Instance tests

The public interface methods should be tested using a shallow renderer and the
`getMountedInstance` helper.

** Example: **

```
describe('SomeComponent/controller', () => {
  function render(props) {
    const renderer = TestUtils.createRenderer();
    renderer.render(<SomeComponentController {...props} />);
    return renderer;
  }

  describe('#componentDidMount', () => {
    it('initializes the data', () => {
      const dataInitializer = sinon.stub();
      const renderer = render({ dataInitializer });
      getMountedInstance(renderer).componentDidMount();
      expect(dataInitializer.args).to.deep.equal([ [] ]);
    });
  });
});
```
