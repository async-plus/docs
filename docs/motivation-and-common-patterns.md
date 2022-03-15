# Motivation and Common Patterns

## Motivation

Async/await is the future of asynchronous coding in Swift. It's missing a few crucial patterns however. Most notably, patterns such as retrying and ensured execution regardless failure status are unwieldy without promises/futures, and catching behavior is much less modular.  Promise-like chaining with Async+ fixes these issues:

### Example: Recovery

```swift
attempt {
    return try await getThing()
}.recover {
    error in
    return try await backupGetThing(error)
}.then {
    thing in
    await thing.doYour()
}.catch {
    error in
    alert(error)
}
```

For comparison, if we tried to write the above flow without Async+ we'd get something like this:

```swift
Task.init {
    do {
        let thing: Thing
        do {
            thing = try await getThing()
        } catch {
            thing = try await backupGetThing(error)
        }
        await thing.doYour()
    } catch {
        error in
        alert(error)
    }
}
```

Async+ allows async and/or throwing code to remain unnested, modular, and concise. 

### Example: Modular failure blocks

Async+ allows us to add catch behavior at any level of a failable operation. For example, we could create methods `printingFailure` and `alertingFailure` as follows, in order to encapsulate different things that we might want to trigger when an error occurs:

```swift
import AsycPlus

extension Catchable {
    /// Prints the error that caused failure
    func printingFailure() -> SelfCaught {
        return self.catchEscaping {
            error in
            print(error.localizedDescription)
        }
    }
    
    /// Displays an alert to the user about the failure
    func alertingFailure() -> SelfCaught {
        return self.catchEscaping {
            alert(error.localizedDescription)
        }
    }
}
```

We can then use `.printingFailure` and `.alertingFailure` for both synchronous and asynchronous chains (corresponding to types `Result<T>` and `Promise<T>`).

For example, using our function `.alertingFailure` synchronously could look like this:
```swift
let result = attempt {
    guard let something = something else {
        throw MockError.notImplemented
    }
    // ...
}.alertingFailure().result
// do something with result
```

Or we could use `.alertingFailure` asynchronously as well:
```swift
let resultInt: Int = try await attempt {
    () -> Int
    guard let something = something else {
        throw MockError.notImplemented
    }
    // ...
    return 0
}.alertingFailure().asyncThrows()
```

!!! note

    You can see we use `catchEscaping` rather than `catch` when passing a non-async closure in a protocol context.  This is the only time that `catchEscaping` should be preferred.  The performance boost using `catchEscaping` is probably negligible though. If you'd like to keep it simple and only ever use `catch`, you would write the extension returning a `CaughtPromise<T>` or a `PartiallyCaughtPromise<T>` as follows:

    ```swift
    extension Catchable {
        func printingFailure() -> CaughtPromise<T> {
            return self.catch {
                error in
                print(error.localizedDescription)
            }
        }
    }

    // OR

    extension Catchable {
        func printingRethrowingFailure() -> PartiallyCaughtPromise<T> {
            return self.catch {
                error in
                print(error.localizedDescription)
    	    throw error
            }
        }
    }
    ```

    The only difference here is that the body passed to `self.catch` is treated as `async`, so a caught or uncaught variant of a `Promise<T>` is always returned.  This means that you will never be able to call `.result` on the output as demonstrated earlier, but will have to use `.asyncResult` instead.

## Common Patterns

How do I...

### Run two chains in parallel?

`async let value1 = attempt{ ... }.async()`

`async let value2 = attempt{ ... }.async()`

You then round up the results by calling `let values = await [value1, value2])`

### Use a chained result with `guard` statements?

```swift
guard let v: Person? = await attempt {
    return try await api.GetPerson()
}.recover {
    return try await localCache.GetPerson()
}.catch {
    error in
    logger.error("We could not get a person")
}.asyncOptional() else {
    
}
```

