# Multiprocessor

## Mp spec

### Ref
- https://pdos.csail.mit.edu/6.828/2012/readings/ia32/MPspec.pdf

### System overview
![](/xv6/notes/image/MP-sys-arch.png)
- The MP specification's model of multiprocessor systems incorporates a tightly-coupled, shared-memory architecture with a distributed interprocessor and I/O interrupt capability. **Fully symmetric, all processors are functionally identical and of equal status, and each processor can communicate with every other processor.**
  - **Memory symmetry**: All processors share the same memroy space and access that space by the same addresses, which offers a very important feature: the ability for all processors to execute a single copy of the operating system.
  - **I/O symmetry**: All processors share access to the same I/O subsystem(including I/O ports and interrupt controllers) and any processor can receive interrupts from any source, helps to eliminate the potential of an I/O bottleneck, thereby increasing system scalability.
- APIC(Advanced Programmable Interrupt Controller)
  - Based on a distributed architecture in wich interrupt control functions are distributed between two basic functional units, local and I/O, which communicate through a bus called the interrupt controller communications(ICC) bus.
  - In mp system, multiple local and I/O APIC units operate together as a single entity, communicate over ICC bus. The APIC units are collectively responsible for delivering interrupts from interrupt sources to destinations through the multiprocessor system.
  - Achieving scalability through:
    - Off-loading interrupt-related traffic from the memory bus, making the memory bus more available for processor used.
    - Helping processors share the interrupt processing load with other processors.
  - Local APIC units also provide interprocessor interrupts(IPIs), which allow any processor to interrupt any other processor or set of processors.
- System Memory
Symmetric mp system imposes a high demand for memory bus banwidth.
- BIOS
A standerd uniprocessor BIOS performs the following functions:
  - tests system components
  - builds configuration tables to be used by the operating system
  - initializes the processor and the rest of the system to a known state
  - provides run-time device-oriented services

And additional functions for mp's BIOS:
  - pass configuration information to the os that identifies all processors and other multiprocessing components of the system.
  - initialize all processors and the rest of the multiprocessing components to a known state

> I will add more details when necessary.