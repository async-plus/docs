# Using chains from async or throwing contexts

## Async contexts, async throwing contexts

Waiting for a chain to complete from within an async or async throwing function is possible as in:

`let value = await attempt{ ... }.async()` 

`let value = try await attempt{ ... }.asyncThrows()`

These value-getting operations are available anywhere in the chain, even after `.finally`.

If the chain doesn't throw you will not be able to call `asyncThrows` on it (it is a `Guarantee<T>` type rather than a `Promise<T>` type), and vice versa.

## Throwing contexts

Additionally, a `throws()` operation exists for non-async throwing contexts.  For non-async chains, no Tasks are created under the hood, and all operations run sychronously.

Here is an example that returns a value using `.throws()`:

```swift
func myThrowingFunc() throws {
    let v = try attempt{ ... }.then{ ... }.recoverEtc{ ... }.throws()
    // Do something with v
}
```
