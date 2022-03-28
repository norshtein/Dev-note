GMP model:  
G - goroutine  
M - machine - 1-to-1 mapping OS thread  
P - processor, queue  

1. How goroutine function invoked?  
G has `sched gobuf` property, in which contains sp and pc. PC is set to goroutine function's address.
2. How goroutine exits?
```go
	newg.sched.sp = sp
	newg.stktopsp = sp
	newg.sched.pc = abi.FuncPCABI0(goexit) + sys.PCQuantum // +PCQuantum so that previous instruction is in same function
	newg.sched.g = guintptr(unsafe.Pointer(newg))
	gostartcallfn(&newg.sched, fn)
```
goexit function's address will be constructed to stack as if goexit calls fn, or its address will be set in `sched.lr` and then put to RA register. When fn returns, it will return to `goexit`, in which `schedule` will be called so that the main loop never exits.
