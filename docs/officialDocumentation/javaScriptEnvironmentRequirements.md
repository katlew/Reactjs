# JavaScript Environment Requirements

* React 16 depends on the collection types Map and Set. If you support older browsers and devices which may not yet provide these natively (e.g. IE < 11) or which have non-compliant implementations (e.g. IE 11), consider including a global polyfill in your bundled application, such as core-js or babel-polyfill.

* A polyfilled environment for React 16 using core-js to support older browsers might look like:

```ts
import 'core-js/es/map';
import 'core-js/es/set';

import React from 'react';
import ReactDOM from 'react-dom';

ReactDOM.render(
  <h1>Hello, world!</h1>,
  document.getElementById('root')
);
```

* React also depends on requestAnimationFrame (even in test environments).
You can use the raf package to shim requestAnimationFrame:

```ts
import 'raf/polyfill';
```