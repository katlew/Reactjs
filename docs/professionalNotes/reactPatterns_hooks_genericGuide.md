# Generic Guides

Here you can find a collection of useful guides how to use hooks.

## How to do side effect on user action

Sometimes there is need to execute some side effect while triggering user action. For example, you need to send request when user press a button. Or more generic, you need to set state inside a callback. If setting state requires the original value of state, the code should use the variant of useState that passes state as argument.

I this case you can do the following thing, example is taken from StackOverflow with some minor changes.

```ts
export const SomeComponent = () => {
  const [isSending, setIsSending] = useState(false);
  const isMounted = useRef(true);
  // set isMounted to false when we unmount the component
  useEffect(() => {
    return () => {
      isMounted.current = false;
    };
  }, []);
  const sendRequest = useCallback(async () => {
    // ⛔ don't send again while we are sending
    if (isSending) {
      return;
    }

    // update state
    setIsSending(true);
    try {
      // send the actual request
      await API.sendRequest();
    } catch (err) {
      // ❗ properly handle error
    }
    // once the request is sent, update state again
    if (isMounted.current) {
      // only update if we are still mounted
      setIsSending(false);
    }
  }, [isSending]); // ✅ update the callback if the state changes
  return <input type="button" disabled={isSending} onClick={sendRequest} />;
};
```

https://smykhailov.github.io/react-patterns/#/generic-hook-guides
