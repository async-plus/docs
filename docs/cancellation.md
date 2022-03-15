# Cancellation

Cancellation is possible through a chained item's exposed `task` property, which returns a `Task<T>` that corresponds to the operation of the chain item AND all previous items in the chain.  This task can then be cancelled using [Swift's standard mechanism for cancellation](https://docs.swift.org/swift-book/LanguageGuide/Concurrency.html#ID642).

You might think that the `task` of a chained item resulting from `catch` is never run, but tasks always runs regardless of success or failure of the chain of operations.

Chained items that are instantaneously evaluated (e.g. `AsyncPlus.Result<T>`) do not have a "task" that can be cancelled, but have a `result` or `value` instead.
