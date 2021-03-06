# Overview

The project is created to experiment with different approaches to use React and create a list of do's, dont's, problematic approaches and best practice in terms of performance, readability, extensibility and maintainability.

# Note about React.memo

Using React.memo is useful for cases where you have a component that should be re-rendered rarely, but is expensive to render. There is still some cost of using React.memo and it might be problematic, if abused in a large app.

# Components

Examples of misusing the components paradigm and how to avoid unwanted re-rendering and make application more robust and maintainable.

# Hooks

A set of techniques to leverage the power of hooks without introducing performance issues.

# Context

Context is powerful concept, but if you don't follow the best practices how to use it, it leads to a massive ammount of the entire components Tree re-rendering. This section describes how to avoid such situations.

# Immutability

It explains an immutability concept and why it is important, when using React.

# Links

The links to the tools, articles about performance and best practices can be found under this section.
