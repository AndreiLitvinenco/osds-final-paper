---
marp: true
theme: beam
paginate: true
header: Virtual machines through the OS lens
footer: Litvinenco Andrei
---
<!-- _class: title -->
# Virtual machines through the OS lens
Andrei Litvinenco
University of Bucharest
6.02.2026

---
# Table of content
- A brief history of virtualisation
- The privilege problem (x86 Rings)
- Strategy 1 - OS level virtualization
- Strategy 2 - Para-virtualization 
- Strategy 3 - Binary translation for unmodified guest OSes
- Strategy 4 - Hardware assisted virtualization
- Classification of hypervisors
- Deep dive into classical virtualization
- The memory challanges in regards with address translation
- Conclusions

---
# A Brief history of virtualization

From Mainframes to Modern Desktops

- The Origins (1960s): Virtualization is not a new concept; it began with IBM's project in 1964 to share the capabilities of physical computers. The "CP/CMS" system defined the term "Virtual Machine," allowing one computer to function as several copies of itself.

- Defining Principles (1974): Researchers Popek and Goldberg established the formal principles of virtualization: Faithfulness, Performance, and Security. They defined a Virtual Machine Monitor (VMM) as a software layer that virtualizes physical machine resources.

---
- The x86 Revival (1999): Interest waned until VMware Inc. introduced the "VMware Virtual Platform" in 1999, bringing virtualization to the standard x86-32 architecture.


- Open Source Innovation (2003): The Xen project, originating from the University of Cambridge, introduced paravirtualization, enabling physical computers to boot multiple operating systems efficiently .


- Hardware Support (2005-2006): Chip manufacturers finally embedded virtualization support directly into processors with Intel VT-x and AMD "Pacifica" extensions, solving many early architectural challenges.

---
# The privilege problem (x86 Rings)

Why Virtualization is Difficult on Standard Hardware

- **Standard Execution Levels**: In a physical computer, the Operating System (OS) operates in Kernel Mode (Ring 0), giving it access to all privileged instructions. User applications run in User Mode (Ring 3) and cannot access privileged instructions directly.

- **The Conflict**: To virtualize a system, the Guest OS expects to run in Ring 0 to control the hardware. However, the Host VMM (Hypervisor) is already occupying Ring 0.

- **The Challenge**: If a Guest OS runs in a less privileged ring (like Ring 1 or 3), it cannot execute necessary privileged instructions. Traditional x86 architecture was not built to facilitate full virtualization because it did not naturally trap these privileged instructions effectively.

---
# Strategy 1 - OS level virtualization

This method virtualizes at the operating system level rather than the hardware level. The host OS kernel is modified to support multiple isolated "containers" (also known as Virtual Private Servers or Jails).
- **Mechanism**: Every container shares the same host operating system kernel. This is different from running a full virtual machine with its own distinct kernel.
- **Pros & cons**:
    - **Advantage**: Implementations like OpenVZ, Solaris Zones, and Linux-VServer exhibit very low overhead.
    - **Disadvantage**: You cannot run different kernels (e.g., you cannot run Windows on a Linux host using this method).

---
# Strategy 2 - Para-virtualization 

Para-virtualization replaces the standard instruction set architecture with a unique set of software instructions known as Hypercalls
- **Execution**: The VMM (Hypervisor) operates directly on the hardware (Ring 0). The Guest OS is modified to recognize it is virtualized and communicates with the VMM via these Hypercalls instead of trying to execute privileged instructions directly.

- **Pros & cons:**
    - **Advantage**: It offers low virtualization overhead because it avoids complex "trap-and-emulate" operations.
    - **Disadvantage**: The Guest OS kernel must be explicitly modified to use Hypercalls, meaning you cannot run unmodified, off-the-shelf operating systems easily.

---
# Strategy 3 - Binary translation for unmodified guest OSes
This strategy allows the execution of unmodified guest operating systems through Binary Translation or full emulation. Esentially running legacy systems via software translation.
- **Full Emulation**: Software like QEMU simulates the entire instruction set of a CPU (e.g., running PowerPC code on an x86 processor). This is flexible but has high overhead.
- **Binary Translation**: Solutions like VMware use a hybrid approach. They allow most user-code to run directly on the CPU for speed. However, privileged code (Ring 0 instructions) is trapped and translated by the VMM into safe code sequences.
- **Performance**: This performs better than full emulation but still incurs overhead compared to running natively. It is strictly limited to guests that match the host architecture (e.g., x86 Guest on x86 Host).

---
# Strategy 4 - Hardware assisted virtualization
Modern processors (Intel VT-x, AMD Pacifica) include hardware extensions specifically for virtualization, eliminating the need for binary translation or paravirtualization.
- **Root** vs. **Non-Root**: These extensions create a new privilege level often called "Root Mode" (Ring -1).
    - **Guest OS**: Runs in Ring 0 (Non-root mode) without modification, believing it has full control.
    - **VMM/Hypervisor**: Runs in Ring -1 (Root mode), intercepting privileged events automatically.

- **Benefits**: This method removes the software overhead of the "trap and emulate" model by handling privileges in hardware. Examples include KVM, VirtualBox, and modern VMware products.

---
# Classification of hypervisors
- **Type I (Native/Bare-Metal)**:
    - **Architecture**: Runs directly above the hardware in the Ring of Highest Privilege.
    - **Function**: It manages memory, CPU scheduling, and I/O directly, without an underlying operating system.
    - **Examples**: VMware ESX, Hyper-V, Xen.

- **Type II (Hosted)**:
    - **Architecture**: Runs inside a standard Host Operating System (like Windows or Linux) alongside other applications.
    - **Function**: It relies on the Host OS process scheduler; every Virtual Machine is just another process to the host.
    - **Examples**: VMware Server, VirtualBox.

---
# Deep dive into classical virtualization
The "Popek and Goldberg" Requirements (1974)
- **Criteria for a VMM**:
    - **Faithfulness**: Software on the VMM must run exactly as it does on real hardware (except for timing).
    - **Performance**: The hardware should execute the vast majority of guest instructions directly, without VMM intervention.
    - **Security**: The VMM must manage all hardware resources.

---
- **De-privileging**: In a classical VMM, the Guest OS is "de-privileged" (moved to a lower ring). When it tries to read/write a privileged state, the CPU "traps" (pauses) and hands control to the VMM.
- **Shadow Structures**: Because the Guest cannot see the real hardware state, the VMM maintains "shadow structures" (like shadow registers). The VMM updates these shadows to trick the Guest into thinking its commands worked.

---
- **De-privileging**: In a classical VMM, the Guest OS is "de-privileged" (moved to a lower ring). When it tries to read/write a privileged state, the CPU "traps" (pauses) and hands control to the VMM.

- **Shadow Structures**: Because the Guest cannot see the real hardware state, the VMM maintains "shadow structures" (like shadow registers). The VMM updates these shadows to trick the Guest into thinking its commands worked.

---
# The memory challanges in regards with address translation
- Software uses "Virtual Addresses" which the hardware (MMU) translates into "Physical Addresses" using Page Tables.

- The Hypervisor must enforce an additional level of translation to isolate guests from each other. The Guest's "Physical Memory" is actually just virtual memory to the Host system.

---
- **Shadow Page Tables**:
    - For unmodified guests, the VMM must fully virtualize x86 paging.
    - The VMM uses "Shadow Page Tables" to map Guest Virtual Addresses directly to Machine Physical Addresses.
    - Tracing: The VMM must "trace" or trap every time the Guest OS tries to update its page tables to keep the shadow tables in sync.


- This creates significant overhead. The CPU's "Page Walker" must traverse memory hierarchies frequently, making address translation a memory-intensive operation.

---
# Conclusions
- **Virtualization** has evolved from a theoretical mainframe concept into a critical technology supporting modern computing. While early methods relied on software tricks like "trap-and-emulate" and binary translation, modern strategies leverage dedicated CPU extensions (Hardware-Assisted Virtualization) for performance and security.
- The choice of virtualization strategy depends on balancing trade-offs:
    - **Paravirtualization**: Reduces complexity and overhead but requires modified Guest OSs.
    - **Full Virtualization**: Allows unmodified legacy OSs to run but requires complex memory management and binary translation, leading to higher overhead.

---

<center><h2>Thank you for your time! 👋​</h2></center>