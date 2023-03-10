# Cancellation Signal

Every Golang command application have at least one goroutine: **the main go routine**, and it is possible to create new goroutines from others.

The resources used by a goroutine are cleaned up when the go routine completes its work or faces an internal error which ends its execution.
In Golang there is no way for one goroutine to force terminating another directly.

When spawning goroutines a good practice is to allow a *parent goroutine* to send a cancellation signal to its *child goroutines* in order to inform them to stop and preventing possibles memory leaks.
