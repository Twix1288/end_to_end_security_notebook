# Week 6 Reflection and Notes

# Overview
I had to change my normal work since my my laptop screen  broke, which made it impossible for me to physically come into the lab, so I spent my time during the lab on the figuring out what parts are missing to make sure the dynamic linking resolution was working.

1. Porting the Dynamic Linking API (dlapi)
I realized that ripgrep and the Rust standard library’s panic handler were not working because they depend on dynamic linking resolution. I had to search it up, but dynamic linking in Rust is the process of linking external libraries at runtime rather than at compile time, allowing executable binaries to "point to" shared library files. I had to go into the dbittman-e1000-driver branch that Daniel told me about to find the __dlapi_* functions and manually move them into my working branch at src/rt/reference/src/syms.rs. I specifically injected:

 - __dlapi_open

-  __dlapi_resolve (dlsym)

-  _dlapi_reverse (dladdr)

-  _dlapi_close, __dlapi_error, and __dlapi_find_object.

I also had to fix a small scope issue by  qualifying twizzler_abi::object::MAX_SIZE to get the symbols to resolve correctly.

2. Managing the POSIX Environment Layer
When I was copying the dynamic linking changes, the upstream merge tried to overwrite the POSIX work I’ve been doing recently. I had to abort the automated merge and had to apply the syms.rs changes myself. This was annoying but had to force myself to do it to ensure my setenv and unsetenv implementations in core.rs stayed intact. I just asked Claude to figure out what things were updated on his branch that I hadn't already tried to do to get ripgrep working and got that on my project.

3. Implementing Human-Readable Backtraces
One of the most satisfying fixes was getting __dlapi_reverse to work. Before this, if ripgrep crashed, the runtime just spit out raw hex memory addresses, which confused the living hell out of me. By getting dladdr working via __dlapi_reverse, the Twizzler runtime can now map those addresses back to actual function names. It’s made debugging actually possible for ocne.

4. Toolchain Recovery
I hit a wall early on because my local macOS build kept failing. It turned out the custom Rust toolchain was corrupted (specifically missing librustc_driver.dylib). Running cargo xtask toolchain pull fixed the environment and let me finally compile the OS with the new integrations.

Challenges & Blockers
I'm still struggling with the fundamental way we handle object IDs. It’s one thing to get symbols resolving, but managing the actual persistent object lifecycle within the driver framework is still hell confusing for me. I’m hoping to sync up with Sean soon—I think he has a better handle on how the object IDs are being mapped in the current runtime, and I could really use his help to make sure I’m not introducing any memory leaks or ID collisions.

Reflection
This week taught me a lot about the "invisible" dependencies in the Rust standard library. I didn't really think about dlopen or dladdr when writing high-level code, but as soon as you move to a custom OS like Twizzler, their absence breaks everything. Even though the screen situation was a massive headache, working remotely forced me to be extremely disciplined with my git branching and manual merges. It's a bit frustrating to be away from the lab hardware, but I feel like I have a much firmer grasp on the linkage between the Twizzler ABI and the Rust runtime now. 
I need to be more proactive about reaching out to the team via Discord when I'm stuck on things like the object IDs, rather than just trying to brute-force it myself.
