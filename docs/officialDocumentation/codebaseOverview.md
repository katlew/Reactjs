# Codebase Overview

This section will give you an overview of the React codebase organization, its conventions, and the implementation.

If you want to contribute to React we hope that this guide will help you feel more comfortable making changes.

We don’t necessarily recommend any of these conventions in React apps. Many of them exist for historical reasons and might change with time.

## Top-Level Folders

After cloning the React repository, you will see a few top-level folders in it:

- packages contains metadata (such as package.json) and the source code (src subdirectory) for all packages in the React repository. If your change is related to the code, the src subdirectory of each package is where you’ll spend most of your time.
- fixtures contains a few small React test applications for contributors.
- build is the build output of React. It is not in the repository but it will appear in your React clone after you build it for the first time.
  The documentation is hosted in a separate repository from React.

There are a few other top-level folders but they are mostly used for the tooling and you likely won’t ever encounter them when contributing.

## Colocated Tests

We don’t have a top-level directory for unit tests. Instead, we put them into a directory called `__tests__` relative to the files that they test.

For example, a test for `setInnerHTML.js` is located in `__tests__/setInnerHTML-test.js` right next to it.

## Warnings and Invariants

The React codebase uses the warning module to display warnings:

```ts
var warning = require("warning");

warning(2 + 2 === 4, "Math is not working today.");
```

**The warning is shown when the warning condition is false.**

- One way to think about it is that the condition should reflect the normal situation rather than the exceptional one.

- It is a good idea to avoid spamming the console with duplicate warnings:

```ts
var warning = require("warning");

var didWarnAboutMath = false;
if (!didWarnAboutMath) {
  warning(2 + 2 === 4, "Math is not working today.");
  didWarnAboutMath = true;
}
```

- Warnings are only enabled in development. In production, they are completely stripped out. If you need to forbid some code path from executing, use invariant module instead:

```ts
var invariant = require("invariant");

invariant(2 + 2 === 4, "You shall not pass!");
```

**The invariant is thrown when the invariant condition is false.**

“Invariant” is just a way of saying “this condition always holds true”. You can think about it as making an assertion.

It is important to keep development and production behavior similar, so invariant throws both in development and in production. The error messages are automatically replaced with error codes in production to avoid negatively affecting the byte size.

## Development and Production

You can use `__DEV__` pseudo-global variable in the codebase to guard development-only blocks of code.

It is inlined during the compile step, and turns into process.env.NODE_ENV !== 'production' checks in the CommonJS builds.

For standalone builds, it becomes true in the unminified build, and gets completely stripped out with the if blocks it guards in the minified build.

```ts
if (__DEV__) {
  // This code will only run in development
}
```

## Flow

We recently started introducing Flow checks to the codebase. Files marked with the @flow annotation in the license header comment are being typechecked.

We accept pull requests adding Flow annotations to existing code. Flow annotations look like this:

```ts
ReactRef.detachRefs = function (
  instance: ReactInstance,
  element: ReactElement | string | number | null | false
): void {
  // ...
};
```

When possible, new code should use Flow annotations. You can run yarn flow locally to check your code with Flow.

## Dynamic Injection

React uses dynamic injection in some modules. While it is always explicit, it is still unfortunate because it hinders understanding of the code. The main reason it exists is because React originally only supported DOM as a target. React Native started as a React fork. We had to add dynamic injection to let React Native override some behaviors.

You may see modules declaring their dynamic dependencies like this:

```ts
// Dynamically injected
var textComponentClass = null;

// Relies on dynamically injected value
function createInstanceForText(text) {
  return new textComponentClass(text);
}

var ReactHostComponent = {
  createInstanceForText,
  // Provides an opportunity for dynamic injection
  injection: {
    injectTextComponentClass: function (componentClass) {
      textComponentClass = componentClass;
    },
  },
};

module.exports = ReactHostComponent;
```

The injection field is not handled specially in any way. But by convention, it means that this module wants to have some (presumably platform-specific) dependencies injected into it at runtime.

There are multiple injection points in the codebase. In the future, we intend to get rid of the dynamic injection mechanism and wire up all the pieces statically during the build.

## Multiple Packages

React is a monorepo. Its repository contains multiple separate packages so that their changes can be coordinated together, and issues live in one place.

## React Core

The “core” of React includes all the top-level React APIs, for example:

```ts
React.createElement();
React.Component;
React.Children;
```

React core only includes the APIs necessary to define components. It does not include the reconciliation algorithm or any platform-specific code. It is used both by React DOM and React Native components.

The code for React core is located in packages/react in the source tree. It is available on npm as the react package. The corresponding standalone browser build is called react.js, and it exports a global called React.

## Renderers

React was originally created for the DOM but it was later adapted to also support native platforms with React Native. This introduced the concept of “renderers” to React internals.

Renderers manage how a React tree turns into the underlying platform calls.

Renderers are also located in `packages/`:

- React DOM Renderer renders React components to the DOM. It implements top-level ReactDOM APIs and is available as react-dom npm package. It can also be used as standalone browser bundle called react-dom.js that exports a ReactDOM global.
- React Native Renderer renders React components to native views. It is used internally by React Native.
- React Test Renderer renders React components to JSON trees. It is used by the Snapshot Testing feature of Jest and is available as react-test-renderer npm package.

* The only other officially supported renderer is react-art. It used to be in a separate GitHub repository but we moved it into the main source tree for now.

## Reconcilers

Even vastly different renderers like React DOM and React Native need to share a lot of logic. In particular, the reconciliation algorithm should be as similar as possible so that declarative rendering, custom components, state, lifecycle methods, and refs work consistently across platforms.

To solve this, different renderers share some code between them. We call this part of React a “reconciler”. When an update such as setState() is scheduled, the reconciler calls render() on components in the tree and mounts, updates, or unmounts them.

Reconcilers are not packaged separately because they currently have no public API. Instead, they are exclusively used by renderers such as React DOM and React Native.

## Fiber Reconciler

The “fiber” reconciler is a new effort aiming to resolve the problems inherent in the stack reconciler and fix a few long-standing issues. It has been the default reconciler since React 16.

Its main goals are:

- Ability to split interruptible work in chunks.
- Ability to prioritize, rebase and reuse work in progress.
- Ability to yield back and forth between parents and children to support layout in React.
- Ability to return multiple elements from render().
- Better support for error boundaries.

While it has shipped with React 16, the async features are not enabled by default yet.

Its source code is located in `packages/react-reconciler`.
