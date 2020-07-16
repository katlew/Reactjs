# Testing Environments

- This document goes through the factors that can affect your environment and recommendations for some scenarios.

## Test runners

- Test runners like Jest, mocha, ava let you write test suites as regular JavaScript, and run them as part of your development process. Additionally, test suites are run as part of continuous integration.

- Jest is widely compatible with React projects, supporting features like mocked modules and timers, and jsdom support. If you use Create React App, Jest is already included out of the box with useful defaults.
- Libraries like mocha work well in real browser environments, and could help for tests that explicitly need it.
- End-to-end tests are used for testing longer flows across multiple pages, and require a different setup.

## Mocking a rendering surface

Tests often run in an environment without access to a real rendering surface like a browser. For these environments, we recommend simulating a browser with jsdom, a lightweight browser implementation that runs inside Node.js.

In most cases, jsdom behaves like a regular browser would, but doesn’t have features like layout and navigation. This is still useful for most web-based component tests, since it runs quicker than having to start up a browser for each test. It also runs in the same process as your tests, so you can write code to examine and assert on the rendered DOM.

Just like in a real browser, jsdom lets us model user interactions; tests can dispatch events on DOM nodes, and then observe and assert on the side effects of these actions (example).

A large portion of UI tests can be written with the above setup: using Jest as a test runner, rendered to jsdom, with user interactions specified as sequences of browser events, powered by the act() helper (example). For example, a lot of React’s own tests are written with this combination.

If you’re writing a library that tests mostly browser-specific behavior, and requires native browser behavior like layout or real inputs, you could use a framework like mocha.

In an environment where you can’t simulate a DOM (e.g. testing React Native components on Node.js), you could use event simulation helpers to simulate interactions with elements. Alternately, you could use the fireEvent helper from @testing-library/react-native.

Frameworks like Cypress, puppeteer and webdriver are useful for running end-to-end tests.

## Mocking functions

When writing tests, we’d like to mock out the parts of our code that don’t have equivalents inside our testing environment (e.g. checking navigator.onLine status inside Node.js). Tests could also spy on some functions, and observe how other parts of the test interact with them. It is then useful to be able to selectively mock these functions with test-friendly versions.

This is especially useful for data fetching. It is usually preferable to use “fake” data for tests to avoid the slowness and flakiness due to fetching from real API endpoints (example). This helps make the tests predictable. Libraries like Jest and sinon, among others, support mocked functions. For end-to-end tests, mocking network can be more difficult, but you might also want to test the real API endpoints in them anyway.

## Mocking modules

Some components have dependencies for modules that may not work well in test environments, or aren’t essential to our tests. It can be useful to selectively mock these modules out with suitable replacements (example).

On Node.js, runners like Jest support mocking modules. You could also use libraries like mock-require.

## Mocking timers

Components might be using time-based functions like setTimeout, setInterval, or Date.now. In testing environments, it can be helpful to mock these functions out with replacements that let you manually “advance” time. This is great for making sure your tests run fast! Tests that are dependent on timers would still resolve in order, but quicker (example). Most frameworks, including Jest, sinon and lolex, let you mock timers in your tests.

Sometimes, you may not want to mock timers. For example, maybe you’re testing an animation, or interacting with an endpoint that’s sensitive to timing (like an API rate limiter). Libraries with timer mocks let you enable and disable them on a per test/suite basis, so you can explicitly choose how these tests would run.

## End-to-end tests

End-to-end tests are useful for testing longer workflows, especially when they’re critical to your business (such as payments or signups). For these tests, you’d probably want to test how a real browser renders the whole app, fetches data from the real API endpoints, uses sessions and cookies, navigates between different links. You might also likely want to make assertions not just on the DOM state, but on the backing data as well (e.g. to verify whether the updates have been persisted to the database).

In this scenario, you would use a framework like Cypress or a library like puppeteer so you can navigate between multiple routes and assert on side effects not just in the browser, but potentially on the backend as well.
