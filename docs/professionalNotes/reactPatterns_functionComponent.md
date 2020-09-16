# Function Component

Function Component - the React.FunctionComponent is the component which takes props and renders them based on internal component logic.

Function Components are functions and not classes. These components use react hooks to control re-rendering and to handle life cycle events.

## Regular Component - Plain props

```ts
// Component is not memoized the parent re-render triggers this component to re-render 💣 too
// even if no props have been changed.
export const ChildFunctionComponent: FunctionComponent<TValue> = (
  props: TValue
) => {
  return (
    <RenderCounter color="blue">
      Child Function Component: {props.value}
    </RenderCounter>
  );
};
```

## Solution: Memoize component with React.memo()

React.memo is a higher order component. It’s similar to React.PureComponent but for function components instead of classes.

If your function component renders the same result given the same props, you can wrap it in a call to React.memo for a performance boost in some cases by memoizing the result. This means that React will skip rendering the component, and reuse the last rendered result.

React.memo only affects props changes. If your function component wrapped in React.memo has a useState or useContext Hook in its implementation, it will still rerender when state or context change.

By default it will only shallowly compare complex objects in the props object. If you want control over the comparison, you can also provide a custom comparison function as the second argument.

```ts
export const ChildFunctionComponentMemoized: FunctionComponent<TValue> = React.memo<
  FunctionComponent<TValue>
>((props: TValue) => {
  return (
    <RenderCounter color="blue">
      Child Function Component <strong> Memoized</strong>: {props.value}
    </RenderCounter>
  );
});
```

## Regular Component - Object props

https://smykhailov.github.io/react-patterns/#/function-component
