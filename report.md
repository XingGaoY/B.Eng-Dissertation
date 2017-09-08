# Syscall Commutativity

## Previous work
Clements, A. T., et al. (2015). "The Scalable Commutativity Rule: Designing Scalable Software for Multicore Processors." Acm Transactions on Computer Systems 32(4): 10.

### Intuition
The state of art for evaluation the scalability of multicore software is to choose a workload, plot performance at varying number of cores and use tools to identify scalability bottlenecks.

However, if there are bottlenecks in more fundamental interface?

**The real bottlenecks in scalable software may be in the interface design.**

So, commuter focuses on software interface, and bottleneck can be solved even before the software implemented.

### Commute
An operation scales if no core writes a cache line that was read or written by another one.  
And adding more cores will produce a linear increase in capacity.
This is called *conflict free*.

Commute: operations in this interface cannot be distinguished by their execution order.  
```
Initial state s0
Two functions(syscall): a(), b()
-----
  r0a = a(s0);  // s0--->s1
  r0b = b(s1);  // s1--->s2
-----
  r1b = b(s0);  // s0--->s1'
  r1a = a(s1'); // s1'--->s2'
```
```
         a         b
    s0 -----> s1 -----> s2
        r0a       r0b

         b         a
    s0 -----> s1'-----> s2'
        r1b       r1a
```
Indistinguishable
```
Execution order a()b() or b()a() is indistinguishable
  Ignore s1 and s1'
  we have:
    s2 = s2'
    r0a = r1a
    r0b = r1b
```
And operations commute have an implementation conflict free.
> As they can be executed in any order, no communication between them is needed
> And eliminating it means no same cache will be visited simutaniously
> which is conflict-free and enhance scalability

### The scalable commutativity rule
Whenever interface operations commute, they can be implemented in a way that scales.

### Commuter
The program in python to test commutativity of FS syscalls(open, link, read ...etc)

According to POSIX interface, commuter simplified the implementation, abstract the data structure and storage, remove some unnecessary value check etc.

And check the commutativity with z3 to solve conditions to find initial state not commute and generate c test code to init the state and invoke syscalls to actully test it in linux.

## Working on
Trying to reimplement commuter to test socket APIs(bind, accept ...etc).
- reimplement commuter
- abstract socket APIs

As I have never studied computer network, I'm reading textbook and implement a simple TCP/IP according to spec and code in linux and lwip.