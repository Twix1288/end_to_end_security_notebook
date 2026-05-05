# Week 5

# What I Did
My technical progress was unfortunately halted this week because my computer screen broke! However, I still managed to make progress on the theoretical side by finishing the written research part of my assigned reading. The paper this week was on Arrakis. Here are my notes on that:

# What is Arrakis All About? (Executive Summary)
Arrakis is a research operating system designed to solve a major bottleneck in modern servers: the operating system kernel. With the invention of incredibly fast hardware (like 10Gb Ethernet and low-latency flash storage/persistent memory), traditional OS kernels (like Linux) have become too slow. In normal systems, the kernel has to mediate every single network and disk operation to enforce security and isolation, which wastes CPU cycles on scheduling, context switching, and copying data back and forth between user space and kernel space.

Arrakis solves this by fundamentally changing the role of the OS. It splits the OS into two parts:

    The Data Plane: Applications are given direct, unmediated access to virtualized network and storage hardware.

    The Control Plane: The kernel is removed from everyday data transfers and acts strictly as a "control plane." Its only job is to set up the hardware, allocate resources, and establish security rules (filters) that the hardware itself enforces.

By letting applications talk directly to the hardware safely, Arrakis dramatically improves performance, demonstrating a 2–5x improvement in latency and a 9x improvement in throughput for popular applications like the Redis NoSQL database.

# Detailed Master Notes

    The Core Problem (OS Overhead): Modern server applications traverse the OS kernel multiple times per client request. Overhead falls into four categories: Network Stack Costs, Scheduler Overhead, Kernel Crossings, and Copying Data. (e.g., In Linux, 76% of the latency for an average read hit in Redis is due to socket operations).

    The Arrakis Architecture & Hardware Model: Arrakis relies on modern hardware virtualization technologies like SR-IOV and IOMMU.

        VNICs and VSICs: Hardware provides Virtual Network Interface Cards and Virtual Storage Interface Controllers, eliminating the need for the kernel to multiplex/demultiplex traffic.

        Hardware Filters: The kernel installs transmit/receive filters to prevent untrusted applications from spoofing or reading other programs' packets.

        Virtual Storage Areas (VSAs): Arrakis replaces the traditional file system with VSAs, a large, persistent segment of physical storage assigned to an application.

        Doorbells: Instead of traditional interrupts, Arrakis uses asynchronous notifications delivered directly from the hardware to the user program.

    Software Abstractions: The actual network and storage stacks are implemented as user-level libraries linked directly into the applications.

        Extaris: Arrakis's custom user-level network stack.

        Caladan: A library of persistent data structures designed for low-latency storage.

    Performance Evaluations & Results: Arrakis reduced server-side processing overhead by 57% using a POSIX interface, and reduced it even further using the native zero-copy interface. It achieved 1.7x the throughput of Linux on Memcached and a massive 9x improvement in write throughput on Redis.

# Troubles
The broken screen completely stopped me from writing code, testing the toolchain, or making any hands-on progress with ripgrep. Hardware failures are the worst!

# Goals and Aspirations
Get my hardware fixed ASAP so I can jump back into the actual porting work, catch up on the ripgrep implementation, and apply what I've learned from Daniel's branch!
