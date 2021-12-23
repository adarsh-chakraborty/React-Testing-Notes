---

# Testing

What actually is testing? We always do the manual testing after implementing the ideas and code, cuz we care what our user sees.

Manual testing is all good but sometimes we might slip some glitches or bugs that ends up in extra work, sometimes we catch it too late or don't catch at all, this is where automated testing comes in.

_Automated Testing is not a replacement of Manual testing but addition. It's a standard thing to do in modern development._

## Kinds of Automated Testing

### Unit Tests

Testing Individual building locks in Isolation. Projects usually contain dozen or hundreds of unit tests. It's most common and important kind of testing.

If you test all the Individual units on themselves, the overall application will also work.

_The Idea of focused tests, "If we have alot of focused tests which can then fail for some clear reason, (if they do) Instead of having large tests which can fail for all different kinds of reasons._

### Integration Testing

To verify everything really works, we test combination of multiple building blocks. Example- Multiple Components working together.

Projects contains a couple of Integration tests but not many as unit tests. It's also important but it's fewer than unit tests.

### End-to-End (e2e) Test

It's all about testing the entire work flow, entire scenerio like logging the user in and then going to a certain page. like in manual testing, just like a real user would do.

We very often wanna test Component than entire e2e because if Unit & Integration tests works then Entire application would also work that is why we don't have as many e2e tests as unit or Integration tests.

So basically e2e tests what we normally do manually anyway so we have fewer of e2e test compared to others like unit or integration.

### What to test

We would test success and error cases that could occur when the user interacts with the application and also test some rare and possible scenerio and results.

## Tools and setup

To get started with testing, we need some extra tools and setup than our application, specifically a tool for running our application and asserting the results.
One of the popular react testing tool is **JEST**

We also need a way of simulating (rendering) our react app/Components. (Simulate the browser)
For this we use, **React Testing Library**.

The amazing thing is both these tools are already setup for us when we create a react app using **create-react-app**.

## Running Tests

To run a test server we can simply run `npm test` to start the testing server which watches all the files for changes and executes `.test.js` files for testing.

## Writing our First Test

First we will create a dummy Component, e.g. Greeting and render it.

```javascript
// Greeting.js
const Greeting = () => {
  return (
    <div>
      <h2>Hello World!</h2>
      <p>It's good to see you!</p>
    </div>
  );
};

export default Greeting;
```

Now let's write test for this spcific Component.

```javascript
// Greeting.test.js
test('Renders hello world as a text', () => {});
```

- `test()` is a global function available to us, we don't need to import it.
- The text string inside the test('<text>') function's first parameter, write something which briefly describe your test.
- The second parameter of test() is a function. We typically wanna do 3 things in here when we write a test.

That is: \*\*The Three "A"s.

1. 1st A stands for, **Arrange** (Setup and render the Component)
2. 2nd A stands for, **Act** (The Logic that should be tested, e.g. execute function on a button click )
3. 3rd A stands for, _Assert_ (Compare execution results with expected results)

So now back to writing our first test,

```javascript
import Greeting from './Greeting';
import {render,screen} from '@testing-library/react';

test('Renders hello world as a text', () => {
    // 1. Arrange
    // Render the greeting component
    render(<Greeting>); // Pass JSX

    // 2. Act
    // .. nothing for now

    // 3. Assert
    // Look in virtual dom and test
    const heloworld = screen.getByText('Hello World' , {exact: true} /* default */); // pass Regular Expression for more flexibility.
    expect(heloworld).toBeInTheDocument(); // pass testing result value i.e. string, number or domNode

    // Pass matcher functions on expect with dot '.'
    // example: .toBeInDocument or opposite with .not.matcher

});
```

There are various methods available on screen, The main difference is when they throw errors and whether they return a promise or not.

- `screen.get..` related methods throws error if element is not found.
- `screen.query..` functions won't do that.
- and `screen.find...` functions returns promise

# Grouping Tests with Test Suites

As our application grows, we will end up of dozens and hundreds of tests, to organize and group such tests easily, so all test belonging to one feature can be grouped into one testing suite.

To create such a testing suite, we use the global `describe()` function, we give it two arguements.

- 1st arguement is a string for category.
- 2nd arguement is a function, but here we don't write testing codes in there instead we put all the test functions inside it.

```javascript
import Greeting from './Greeting';
import {render,screen} from '@testing-library/react';

describe('<Greetings /> Suite', () => {
    test('Renders hello world as a text', () => {

    render(<Greeting>);
    const heloworld = screen.getByText('Hello World' , {exact: true} );
    expect(heloworld).toBeInTheDocument();
    });
});
```

# User Interaction and State

While writing tests, we wanna test all scenerios otherwise our test aren't worth that much.

Let's have a state in the component, and import userEvent for click.

```javascript
import { useState } from 'react';

const Greeting = () => {
  const [changedText, setChangedText] = useState(false);

  const changeTextHandler = () => {
    setChangedText((prev) => !prev);
  };

  return (
    <div>
      <h2>Hello World!</h2>
      {!changedText && <p>It's good to see you!</p>}
      {changedText && <p>Changed!</p>}

      <button onClick={changeTextHandler}>Change</button>
    </div>
  );
};

export default Greeting;
```

Here, we would write two tests for each scenerio

```javascript
import { render, screen } from '@testing-library/react';
import userEvent from '@testing-library/user-event';
import Greeting from './Greeting';

test('Greeting test change', () => {
  // Arrange
  render(<Greeting />);

  // Act
  // ... nothing
  // Assert
  const p = screen.getByText("It's good to see you!", { exact: false });
  expect(p).toBeInTheDocument();
});

test('render change if button clicked', () => {
  //Arrange
  render(<Greeting />);

  // Act
  const buttonElement = screen.getByRole('button');
  userEvent.click(buttonElement);

  // assert
  const p = screen.getByText('Changed!');
  expect(p).toBeInTheDocument();
});

test('does not render good to see you if button clicked', () => {
  //Arrange
  render(<Greeting />);

  // Act
  const buttonElement = screen.getByRole('button');
  userEvent.click(buttonElement);

  // assert
  const p = screen.queryByText('good to see you', { exact: false });
  expect(p).toBeNull();
});
```

We used `screen.queryByText` because `getByText` would throw an error immidiately.

## Testing Connected Components

If one component contains more Components inside it, `render()` function will simply render the entire component tree and would not throw an error.

For Example, If **B** is a component which is rendered inside component **A**.

We only have to render component **A** as component **B** will be rendered inside/with **A** when `render()` is called.

It doesn't require us to split our test code for different Components.

_Testing conjunction of two or more Components comes under Integration testing, as we learned._

## Testing A-Synchronous Code

Suppose we are fetching some data from an API with useEffect hook.

```javascript
import ASync from './ASync';
describe('Async component', () => {
  test('renders post if req succeeds', async () => {
    render(<ASync />);

    const listItemElements = await screen.findAllByRole('listitem');
    // .findAllByRole('listitem', {exact: true}, {timeout: 1000});
    expect(listItemElements).not.toHaveLength(0);
  });
});
```

- Instead of using `getAllByRole` we used `findAllByRole` because it returns a promise.
- We used Async keyword in function, so we can await the promise to be fullfilled.
- `findAllByRole` can also have a third arguement, to set the timeout, default is 1 second.

## Mock Tests

Currently, we are testing our application which actually sends a http request to a server, which is not Ideal for testing environment for various reasons.

- It causes unnecessary traffic to our servers.
- If we are sending a post request, it will insert the data into our database.
- Too many of these requests suppose we have hundreds of test cases, so rip.
- Tests would fail if our backend API server is offline or not working as expected.

So basically, we don't wanna send requests in testing which changes things in database.

What can we do now, We can either not send request at all or we do it on some kind of fake server like a testing server.

For example, we want to check how our Components behaves on response or errors from fetch, not testing actual fetch function to send request that is built-in on browsers. We rely on the browser for fetch function to work correctly.

To write such mock fetch function

```javascript
import ASync from './ASync';
describe('Async component', () => {
  test('renders post if req succeeds', async () => {
    // In the arrange
    window.fetch = jest.fn();
    window.fetch.mockResolvedValueOnce({
      json: async () => [
        {
          id: 'p1',
          title: 'First post'
        }
      ]
    });
    render(<ASync />);

    const listItemElements = await screen.findAllByRole('listitem');
    // .findAllByRole('listitem', {exact: true}, {timeout: 1000});
    expect(listItemElements).not.toHaveLength(0);
  });
});
```

**Explaination:**

- Replacing `window.fetch` with a dummy mock function provided by `jest`.
- We can call some special methods on that function, here we called `mockResolvedValueOnce` to provide a value when it resolves.
- Here we tried to replace browser fetch function with a mock fetch function which has a `json` key which holds a function and returns data when executed.
- It will work same as fetch but here we aren't sending any http requests to servers.

### More Resources

- Check documentation of Jest, React testing library, react hooks testing library.
