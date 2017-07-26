# Abstract
- Problem  
Verification of fs to form a bug-free file fs in the presence of **system crash** and **reordering of disk operations**.
- Main method  
  - Crash refinement
  - SE: exhaust all the posible paths against all the disk states
    - partition the problem into multiple layers to avoid path explosion  
    > first, all the paths in this module, whatever the input is, have been exhausted, and no corner state exists.
    > this module can be used as a fun
    - Why "if an implementation is cr of an spec, they are **indistinguishable** to higher layers"
- Input
  - specification of the expected behavior
    - data structure & precondition of the data structure**(it is analogous to the consistency invariant mentioned later)**
    - file system operations
    - state equivalence predicate: what it means for a given fs state to be **correct**(maybe not cons-invar?)
  - implementation
    - disk model: with disk operations
    - log-structured fs
    - the example: 
      - all the operation is of a sole call
      - operations are free to exchange
      - if system crush before 4 after 5, inconsistency happens.
      - a flush? is needed to keep operations in order expected as a mb?
  - **consistency invariants** indicatng the fs in a consistent state
    - **analogous to the well formedness invariant for its spec**
    - to determines whether a given disk state corresponds to a valid fs img.
    - in YminLFS, three ~loose~ constraint is used on different components on disk
- Crush refinement
  - \mathscr{I}: the consistency variant
  - impl F_1 is correct with respect to spec F_1
  > That is, prove the impl to be right based on the spec pre-defined
    - starting form equivalent consistent states and invoking the same operation on both systems, any state produced by F_1 is equivalent to some state produced by F_0
    > in which aspect do these two system different? the impl is more detailed and complex?
  - system operations
    - function with inputs:
      - current state
      - external input
      - crash schedule: boolean denote the occurence of crash events
  - ec: equivalent and consistent
  - crash-free equivalence
    - crash schedule to be true, s.ec implies f(s).ec
    - here f_0 & f_1, I guess, is the same operation in spec and actual implementation
  - crash refinement without recovery
    - crash schedule
    - NOTICE:: **Forall b_1, Exist b_0**, it is a existence predicate here
      - thus spec(b_1) can be viewed as a subset of b_0, in some regards
      > here spec(b_1) <= equavilent b_1 in spec
  - crash refinement with recovery
    - recovery operation is idempotent, r() won't change the state recovered.
    - a recover operation is now operate on the impl fun
  - no-op
    - background operations do not change the externally visible state of a system
  > And now the question come up: what is b, the crash schedule?
  > only a pair (on, sync) is not a well definition of cs
  > and according to the definition, b in spec and impl can be different when the states and operations are equivalent? How does it come?
  > It is easy to solve this question by examine how b plays a part in f
  > 3.3 is important
