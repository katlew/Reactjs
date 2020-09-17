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

Passing objects as properties to the component are dangerous in terms of causing not wanted re-rendering. If component needs to work only with subset of the object properties and none of them being changed, the component still might re-render if any of the other property has changed.

Also, even if developer created the object and passes it as a parameter to the component, it doesn't prevent other developers to add their own properties to the same object without even knowing that it might have negative impact on re-rendering some other not related component.

```ts
export type TObjectProps = {
  obj: TObjectValue;
};

export type TObjectValue = {
  num: number;
  str: string;
};
```

The component takes object properties as defined above:

```ts
export const ChildFunctionComponentWithObjectProps: FunctionComponent<TObjectProps> = (
  props: TObjectProps
) => {
  // The component only works with ✅ obj.str property and ignores ✅ obj.num
  // If parent component doesn't change the ✅ obj.str, but changes ⛔ obj.num
  // this component will still re-render 💣
  return (
    <RenderCounter color="blue">
      Child Function Component: {props.obj.str}
    </RenderCounter>
  );
};
```

When parent component changes obj.num and doesn't change obj.str, the component still re-renders.

## Solution 1: Use plain props

```ts
// ✅ Memoize component to make sure it doesn't re-render, when props have been changed.
export const ChildFunctionComponentMemoized: FunctionComponent<{
  str: string;
}> = React.memo<Function<{ str: strings }>>((props: { str: strings }) => {
  return (
    <RenderCounter color="blue">
      Child Function Component <strong>Memoized</strong>: {props.str}
    </RenderCounter>
  );
});
```

## Solution 2: Add comparison function

```ts
export const ChildFunctionComponentWithObjectPropsMemoized: FunctionComponent<TObjectProps> = React.memo<FunctionComponent<TObjectProps>>(
  (props: TObjectProps) => {
    return (
      <RenderCounter color = "blue">
        Child Function Component <strong> Memoized</strong>: {props.obj.str}
      </RenderCounter>
    );

  },
  // ✅ Add the properties comparison function.
  {
    (prevProps: Readonly<TObjectProps>, nextProps: Readonly<TObjectProps>) => {
      return prevProps.obj.str === nextProps.obj.str;
    }
  }
);
```
