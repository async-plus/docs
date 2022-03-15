# FAQ

***Why am I getting an unused value warning from Xcode?***

Chains that allow failures to go uncaught (without `.catch` or `.recover`) will raise an "unused result" warning at compile time if they are not assigned to a variable and used. Similarly, unused results returned from `then` or `recover` will raise a warning.

***What is the difference between the `ensure` operation and using Swift's built-in `defer`?***

Swift's `defer` will execute the provided block at the end of the current scope. This is different from `ensure`, which executes regardless of success or failure _at the place that it is put in the chain_. Although `defer` preserves order of execution with regard to other `defer` statements, this order is not in relation to the surrounding `await` operations like `ensure` is. Additionally, if a function throws before `defer` is called, then the block will not be run at all.  Even though at first glance they may be used for similar things (e.g. cleaning up of resources), they behave quite differently with respect to when and if they are run.
