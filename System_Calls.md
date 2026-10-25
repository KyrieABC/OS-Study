# System Call
  - Services offered by the kernel to user applications

## Linux System Calls implementation
  - System call is not the typical function, but rather **specific assembly instructions** that does:
    1. Setup information to identify the system call and its parameters
    2. Trigger a kernel mode switch
    3. Retrieve the result of a system call
  - In Linux, system calls (identified by numbers)
    - parameters for system calls are machine word sized (32 or 64 bit)
    - at max 6 system call parameters (Ex: in 32 bit x86 architecture, system call identifier is stored in EAX register, parameters stored in EBX, ECX, EDX, ESI, EDI, EBP register)
      - x86 is a CPU architecture (all registers are within CPU: fast storage locations that holds temporary data, instructions, moemory addresses currently used)
    - System libraries offers functions that implement actual system calls
    - When a transition from user to kernel mode occurs, execution flow is **interrupted** and it is transferred to a kernel entry point (similar to how interrupts and exceptions are handled)
    - system call entry point will save registeres (contains values from user space, including system call number and system call parameters) on stack then continue with system call dispatcher
      - Purpose of system call dispatcher is to verify system call number and run the kernel function associated with that number
      - During user-kernel mode transition, the stack also switched from user stack to kernel stack
    ```dup2 system call lifecycle
    Application:
      initiate system call through standard system libraries (Ex: libc)
      - C library prepares arguments and the system call number by moving them into specific CPU register
    Transition to Kernel (Trap):
      - libc stub executes an assembly instruction to trigger a software interrupt or trap 
      - CPU execution mode immediately swtich from User Mode to Kernel mode, jumpt to a pre-defined kernel entry point
    -> Kernel (privileged kernel space where CPU executes OS code to handle the request)
      - kernel entry point saves user-space registers and user stack information onto a protected kernel stack (info won't lose when application left)
      - (Dispatcher) kernel calls a dispatcher function that reads system call number from eax and executes underlying kernel logic for that request
    Return to User space:
      - Once kernel operation finishes, it restores saved-user-space registers from stack
    ```

## System Call Table
  - what system call dispatcher uses to map system call numbers to kernel functions

## System Call Parameter handling
  - values are setup by user space, kernel can not assume correctness, must verify them
  - Pointers: 
    - **Never allow pointers to kernel space** (avoid user space from getting access to kernel space)
      - `write()`: kernel needs to read data out of the pointer(if point to kernel space, user program could get access to kernel space-where sensitive OS configuration, passwords, other users data reside)
      - `read()`: kernel will write incoming data into pointer (if point to kernel space, program could corrupt OS internal state which might leads to system crash)
    - check for invalid pointers (bad pointer: If kernel tries to read or write direclty to user-memory space address using a raw pointer, it risks hitting an invalid, unmapped, or swapped-out address -> instant panic and crash the kernel)
  1. Check pointer against the user address space before using it
  2. Avoid checking the pointer and rely on the MMU to detect when pointer is invalid, use the page default handler to determine that the pointer was invalid
    - page default handler (only sees) use fault address (the one that was accessed), the faulting address (the address of instruction that did the access) and information from the user address space to determine the cause:
      1. **Normal User Operation (valid)**: Fault address(user space), Faulting address(user space) Ex: copy-on-write, demand paging, swapping
      2. CPU executing valid kernel code (handling of system call), but fault address is a bad or kernel-space pointer
      3. CPU is executing kernel code, and a bug inside the kernel code itself accidentally tried to dereference a bad or kernel-space pointer
        - Page Default handler views 2 and 3 as the same
    - (in order to resolve that) Linux uses special APIs to accesses user space
      - Exact instructions that access user space are recorded in **exception table**, when a page fault occurs, the faulting address is checked against this table (distinguish between 2(user space system call parameter passing - trace back to here) and 3(kernel internal error))
      - Verify the pointer actually resides in valid user space (no tin protected kernel memory)
  
## Virtual Dynamic Shared Object (VDSO)
  - optimizing system call implementation in a way that does not impact libc with having to track the CPU capabilities in conjunction with the kernel version
  - Transition from User to Kernel(trigger a system call) mode uses `int 0x80`(software interrupt, slow due to triggering a heavy software interrupt gate in CPU) or `syscenter/syscall` (Special hardware instructions introduced in newer CPUs, does transition without overhead of an interrupt gate)
    - TO determine to use which (`libc`):
      1. `libc` check CPU features too see if processor supported `syscenter`
      2. `libc` check if running Linux version supports `syscenter`
      - Every time a new CPU instruction or kernel optimization came out, every single version of `libc` had to be updated and re-compiled
    - (VDSO) Instead of `libc` guessing what hardware and kernel do, the kernel itself decides and builds the code
      1. When Linux kernel boots up, it dynamically generates a tiny piece of machine code in its own memory (code(formatted as a valid ELF shared object) contains exact, optimal sequence of instructions needed to perform a system call for that specific running combination of CPU and kernel)
      2. When a new process is created (`fork()` and `exec()`), kernel takes the internal ELF shared object and maps a read-only copy of it directly into the process's address space (directly into user space so user mode code can read and execute it without needing a system call)
      3. When `libc` initializes itself when a program starts up, it doesn't hardcode CPU checks, instead it looks at process's auxiliary vector to find a special pointer `AT_SYSINFO_EHDR` (if present, `libc` knows the kernel has provided a VDSO)