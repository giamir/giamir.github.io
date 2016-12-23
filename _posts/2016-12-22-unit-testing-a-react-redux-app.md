---
layout: post
permalink: /unit-testing-a-react-redux-app
title: 'Unit Testing a React Redux App'
path: _posts/2016-10-22-unit-testing-a-react-redux-app.md
time: 10
---

Redux is a tool for managing data-state and UI-state in JavaScript applications.
Features like having one store as the single source of truth, as well as actions and reducers that consist of pure functions made this tool so popular in the JS community especially when it is used in combination with React.

One of the last challenge we faced on the project I currently work on was too find out a strategy to structure our React Redux unit tests.

## Why unit testing is important?
Before starting any sort of rambling it is good to stop for one second and remind ourselves of the benefits unit testing provides.

> Unit testing is a type of automated test that is the closest to the code itself.

Unit tests help find software bugs early; they give confidence to devs when it is refactoring time (hence always) and they provide live documentation to our software. They are also the bedrock of **T**est **D**riven **D**evelopment.

> Unit tests provide **fast** feedback on code.

These are just some of the reasons why if you are not unit testing your React Redux application yet you should start to think of doing so.

## How to unit test a React Redux Application?
React and Redux are pretty new technologies and, as always, in software development there is no absolute dogmatic way to test code; developers take different approaches and use different tools based on their needs.

After a few weeks of research me and my team came up with a *React Redux Manifesto*.

![React Redux Manifesto](https://s3.eu-west-2.amazonaws.com/websitegiamir/react-redux-manifesto.png)

It was a way to start being consistent within the team. We collected testing guidelines and we explained why we decided to test things in a way rather than another one.
Today I want to share our takeaway from this experience, hoping this could help other developers to shape a testing strategy for their React Redux applications.

Enough talking, let's dive into the matter.

First of all we need to understand how the architecture of the app looks like, so that we can decide what we should test. In the project I'm currently working on we mainly use 2 patterns to organise our frontend code:

* The Redux design pattern
* The Presentational and Container components pattern

### The Redux design pattern

It's not the purpose of this post to explain how Redux works, there are already gazillions of good resources for that (some of them are in the further reading section).

Anyway it is useful to quickly refresh how the state is managed in a Redux application.

![Redux Design Pattern](https://s3.eu-west-2.amazonaws.com/websitegiamir/redux-design-pattern.png)

These are the 3 Redux principles:

* The state of the application is stored in an object tree within a single store. (Single source of truth)

* The only way to change the state is to emit an action, an object describing what happened. (State is read-only)

* To specify how the state tree is transformed by actions we use pure reducers. (Changes are made with pure functions)

Our UI components (more specifically our container components), *subscribe* to the store and they are able to *dispatch* actions to it.

Let's keep this in mind and look at how we integrate React with the Redux design pattern.

### The Presentational and Container components pattern

This is a simple pattern you can use when writing React applications. You have probably already heard about it if you are familiar with React. Dan Abramov (the creator of Redux) explains it very well in [this article](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.jkzbxbnf8).

Let's quickly refresh what Container components (aka Smart) and Presentational components (aka Dumb) should be responsible for in a React app.

![Presentational and Container components pattern](https://s3.eu-west-2.amazonaws.com/websitegiamir/presentational-container-components.png)

#### Presentational components

* Are concerned with how things look.
* Often allow containment via `this.props.children`.
* Have no dependencies on the rest of the app, such as actions or stores.
* Don’t specify how the data is loaded or mutated.
* Receive data and callbacks exclusively via props.
* Rarely have their own state.
* Are usually written as functional components.

#### Container components

* Are concerned with how things work.
* Provide the data and behaviour to presentational or other container components.
* Call actions and provide these as callbacks to the presentational components.
* Are often stateful, as they tend to serve as data sources.
* Are usually generated using higher order components such as `connect()` from React Redux.


### The final architecture, what should I test?

If we connect the dots this is the picture of our final architecture we want to test.

![Final Architecture](https://s3.eu-west-2.amazonaws.com/websitegiamir/final-architecture.png)


I will use an [example](https://github.com/giamir/unit-testing-react-redux) to better explain the decision I took to test the application.

The example made use of [Webpack](https://webpack.github.io/) to bundle the app modules and to use [React Hot Reloading](https://medium.com/@dan_abramov/hot-reloading-in-react-1140438583bf#.1k9xqc8h9). [Jasmine](https://jasmine.github.io/) with [Karma](https://karma-runner.github.io/1.0/index.html) is used to write and run the tests.<br>
I also used [Airbnb Enzyme](http://airbnb.io/enzyme) testing utility for React together with the [Enzyme matchers](https://github.com/blainekasten/enzyme-matchers) assertions library to make our tests and assertion more readable and easy to understand.

*DISCLAIMER: What follow are personal opinions on how to test React and Redux, they are not the right way but instead what turned out to be effective for me and my team.*

### Presentational components

![Presentational Components](https://s3.eu-west-2.amazonaws.com/websitegiamir/presentational-components.png)

I found presentational components the easiest to test. They do not contain any sort of logic and they are often the leaves of the React DOM tree which means they often do not have children.

Let's take a simple `Button` component as an example:

``` js
const Button = props => (
  <a onClick={props.onClick} className='button'>
    {Children.toArray(props.children)}
  </a>
);
```

#### Rendering tests
Although it can seems pretty dumb, it makes total sense to test if the component did render as expected.
Let's say that check if the rendering works fine is the starting point when we test components.

``` js
const children = 'Save';
const renderComponent = (props = {}) => mount(
  <Button {...props}>
    {children}
  </Button>
);

describe('Button <Button />', () => {
  it('should render the button', () => {
    const renderedComponent = renderComponent();

    expect(renderedComponent).toBePresent();
  });

  it('should render the children', () => {
    const renderedComponent = renderComponent();

    expect(renderedComponent).toHaveText(children);
  });
});
```

#### Event callbacks tests
For presentational components another important bit to test are **callbacks**.<br>
In this example we want to check that when I click the button the `onClick` callback function (a Jasmine spy in our test) I passed as a prop to the `Button` component has been called correctly.

``` js
it('should handle click events', () => {
  const onClickSpy = jasmine.createSpy('onClickSpy');
  const renderedComponent = renderComponent({ onClick: onClickSpy });
  renderedComponent.find('a').simulate('click');

  expect(onClickSpy).toHaveBeenCalled();
});
```

### Container components

![Container Components](https://s3.eu-west-2.amazonaws.com/websitegiamir/container-components.png)

When we talked about the _presentational container components pattern_ we pointed out that container components are usually generated using higher order components. In our case we are using `connect()` from the [React Redux](https://github.com/reactjs/react-redux) library to be able to inject Redux state into a regular React component.

Let's take this simplified version of the `App` component:

``` js
export class App extends React.Component {
  render() {
    return (
      <div>
        <h1>Github Profiles</h1>
        <List items={this.props.users} />
        <Button onClick={this.props.onClickButton}>Add</Button>
      </div>
    );
  }
}

export function mapDispatchToProps(dispatch) {
  return {
    onClickButton: () => dispatch(loadUserInfo())
  };
}

const mapStateToProps = createStructuredSelector({
  users: selectUsers()
});

export default connect(mapStateToProps, mapDispatchToProps)(App);
```

<small>[Complete version of the App component](https://github.com/giamir/unit-testing-react-redux/blob/master/app/containers/App/index.js)</small>

In order to be able to test the App component itself without having to deal with the decorator `connect([...])(App)`, _Redux_ documentation recommend to also export the undecorated component `App`.

We are also exporting the `mapDispatchToProps` function in order to test it.

> <small>It's interesting to observe we decided to do not export `mapStateToProps` for testing purposes.
This is because we are using a library from Facebook called [Reselect](https://github.com/reactjs/reselect) to create memoized, composable selector functions.
The aim of this article is not to explain how _Reselect_ works but keep in mind that if you do not decide to use selector functions (with or without _Reselect_) you should also export `mapStateToProps` and test any logic in it.
If you are interested to see how the selectors code looks like in the example you can find it [here](https://github.com/giamir/unit-testing-react-redux/blob/master/app/containers/App/selectors.js) and the related tests [here](https://github.com/giamir/unit-testing-react-redux/blob/master/app/containers/App/specs/selectors.specs.js).</small>

#### Rendering tests
Again check if the rendering works fine is the starting point when we test components.
In this case because our container component has several children we want to check in our tests that the `renderedComponent` (Enzyme wrapper) contains every React component instance which is supposed to be initialised given a determined set of props.

``` js
describe('<App />', () => {
  const props = {
    users: []
    onClickButton: () => {}
  };

  it('should render the heading', () => {
    const renderedComponent = shallow(<App {...props} />);

    expect(renderedComponent).toContainReact(<h1>Github Profiles</h1>);
  });

  it('should render the list of items', () => {
    const renderedComponent = shallow(<App {...props} />);

    expect(renderedComponent).toContainReact(<List items={props.users} />);
  });

  it('should render the add button', () => {
    const renderedComponent = shallow(<App {...props} />);

    expect(renderedComponent).toContainReact(
      <Button onClick={props.onClickButton}>Add</Button>
    );
  });
});
```

> <small>Although it makes sense to test if React is rendering the children of a container component given a determined set of props, it is questionable to check the attributes of the rendered components because they are details which tend to change very quickly.
So in case we decide to do not test them in the example above our tests would change and use the `toBePresent` assertion rather than the `toContainReact` one.</small>

#### Redux connections tests

In `mapDispatchToProps` we are only composing action creators with dispatch under regular circumstances.

In the test below we are checking the object returned by the `mapDispatchToProps` function contains an `onClickButton` property. We are also testing the `dispatch` function has been called with the action object returned by the `loadUserInfo` action creator whenever the `onClickButton` property has been called.

``` js
describe('mapDispatchToProps', () => {
  describe('onClickButton', () => {
    it('should be injected', () => {
      const dispatch = jasmine.createSpy('dispatch');
      const result = mapDispatchToProps(dispatch);

      expect(result.onClickButton).toBeDefined();
    });

    it('should dispatch addItem when called', () => {
      const dispatch = jasmine.createSpy('dispatch');
      const result = mapDispatchToProps(dispatch);
      result.onClickButton();

      expect(dispatch).toHaveBeenCalledWith(loadUserInfo());
    });
  });
});
```

> <small>Someone can argue this tests are dumb under regular circumstances. Anyway we found this tests useful to catch errors when action creators are assigned to the wrong prop by mistake.</small>

### Actions
![Actions](https://s3.eu-west-2.amazonaws.com/websitegiamir/actions.png)

In Redux, action creators are functions which return plain objects.

``` js
export function changeTextField(value) {
  return {
    type: CHANGE_TEXT_FIELD,
    value
  };
}
```
Because action creators are just wrapper around action object we decided to do not test them.
Besides by creating the actions via action creators when testing reducers, we’re already verifying that the action creators work as expected.

### Reducers
![Reducers](https://s3.eu-west-2.amazonaws.com/websitegiamir/reducers.png)

Changes are made with pure function in Redux and the reducer is a pure function that takes the previous state and an action, and returns the next state.

In this example we use _ImmutableJS_ to manipulate the state of our application.

> <small>ImmutableJS is a library for creating collections of data, which, once created, cannot be changed. These collections are modelled on JavaScript’s Array, Map and Set objects, but with the significant difference that all methods to add, delete or update data in a collection do not mutate the collection being acted upon. If you want to expand on how ImmutableJS works you can checkout the [official documentation](https://facebook.github.io/immutable-js).</small>

Let’s take this simplified version of the `appReducer`.
Here the reducer is responsible only to handle actions of type `CHANGE_TEXT_FIELD` that will return a new state with a `textField` value equals to `action.value`. In all the other cases (default) it returns the previous state.

``` js
const initialState = fromJS({
  textField: ''
});

function appReducer(state = initialState, action) {
  switch (action.type) {
    case CHANGE_TEXT_FIELD:
      return state.set('textField', action.value);
    default:
      return state;
  }
}
```

<small>[Complete version of the appReducer](https://github.com/giamir/unit-testing-react-redux/blob/master/app/containers/App/reducer.js)</small>

Testing reducers should be quite simple because they are pure functions (same data in, same data out, no side effects). As I mentioned before another thing to consider is to use action creators when creating the action for the reducer under test. Taking this approach allows us to avoid explicitly testing any action creators.

<small>In the following tests we are using the [Jasmine Immutable Matchers](https://github.com/unindented/custom-immutable-matchers) library to be able to use the `toEqualImmutable` assertion on Immutable objects.</small>

``` js
describe('appReducer', () => {
  let state;

  beforeEach(() => {
    state = fromJS({
      textField: ''
    });
  });

  it('should return the initial state', () => {
    const expectedResult = state;

    expect(appReducer(undefined, {})).toEqualImmutable(expectedResult);
  });

  describe('changeTextField action', () => {
    it('should change the textField with the action value', () => {
      const value = 'a value';
      const expectedResult = fromJS({
        textField: 'a value'
      });

      expect(appReducer(state, changeTextField(value))).toEqualImmutable(expectedResult);
    });
  });
});
```

> <small>In the case you decide to do not use ImmutableJS you need to take into account that the previous state can mutate when is passed to the reducer as argument. You need to write test that consider this eventuality and you may want to `Object.Freeze` recursively your state before passing it to the reducer using a library such as [deepFreeze](https://github.com/substack/deep-freeze).</small>

### End to End tests

![End to End tests](https://s3.eu-west-2.amazonaws.com/websitegiamir/end-to-end.png)

I don't want to treat E2E tests in depth here.
I just want to point out that although mainly tools like Selenium, Protractor, etc are used to do E2E testing the nature of React Virtual DOM allow us to mount the root component of our application and simulate user interactions using a testing utility such as Enzyme.

We have a very fast and convenient way to test the most critical part of the application thanks to React, let's make sure to use it and consolidate the bedrock of our [test pyramid](http://martinfowler.com/bliki/TestPyramid.html).

> <small>You will always need some functional Selenium tests in your application, but you can write a great part of them in a more unit level fashion and cut the final cost of your test suite.</small>


## Wrapping up

How to unit test a React Redux Application? What should I test?

* **Presentational components**, test rendering and event callbacks
* **Container components**, test rendering and Redux connections
* **Actions**, test they work as expected in the reducer tests
* **Reducers**, test all the logic and check they are pure (don't mutate the state)
* **End to End tests**, render the root component and simulate user interactions

In this article I did not consider testing strategies for __Redux middlewares__.
I'm planning to treat this topic in a separate blog post.

> __1001 ways to manage side effects in Redux__

Well, if you are still with me at this point of the post you surely are a tough person and I would be happy to hear what is your way to unit test a React Redux application. Drop me a comment below!

You can find the entire source code of the examples above in this [Github repo](https://github.com/giamir/unit-testing-react-redux).

### Further reading

* [Redux Documentation: Writing Tests](http://redux.js.org/docs/recipes/WritingTests.html)
* [Some Thoughts On Testing React/Redux Applications](https://medium.com/javascript-inside/some-thoughts-on-testing-react-redux-applications-8571fbc1b78f#.npx2r82zx)
* [Worthwhile Testing: What to test in a React app (and why)](https://daveceddia.com/what-to-test-in-react-app/)
* [Getting started with Redux](https://egghead.io/courses/getting-started-with-redux)
* [Presentational and Container components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0#.jkzbxbnf8)
