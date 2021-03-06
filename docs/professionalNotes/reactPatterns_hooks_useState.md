# useState hook

## Calculate state based on initial value

Common source of bugs in React is when the code inside a callback computes a new value of state that needs to use the previous state value. In that case, the code needs to use the variant of the setState function that receives the current state value as parameter.

```ts
const source = "Some string";
const value = React.useRef<string | undefined>(source);
const [state, update] = React.useState({
  stringProp: "string prop value",
  boolProp: false,
  hasValue: !isEmptyValue(source),
});

const onChange = React.useCallback(
  (newValue: string) => {
    value.current = newValue;
    const newHasValue = !isEmptyValue(newValue);

    // It is important ❗ to do the value comparison inside the update function
    // to ensure that the callback being called by react is the updated
    // once referencing the new state.
    update((state) =>
      newHasValue !== state.hasValue
        ? { ...state, hasValue: newHasValue }
        : state
    );
  },
  [update]
);
```
