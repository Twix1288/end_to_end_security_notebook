# Week 7 Reflection
# What I Did This Week
This week's work focused on leveling Twizzler's init shell to make rg usable:
- double-quote parsing: added a shell_split() to the init shell to support argument parsing. Allows commands like rg "fn main" /initrd/ to be passed, grouping quoted strings as a single argument.
- faking a directory: rg calls getcwd() on startup to determine its search root. To handle this, I updated to automatically inject a PWD=/ environment variable into all new compartments, giving ripgrep a valid root directory to work from -> Testing for right now

# CHERI Research Discussion
We discussed the CHERI architecture research paper, covering:

- How bounds work in CHERI — capabilities encode bounds directly alongside the pointer, so the hardware can enforce memory access limits at runtime.
- Similarities to Rust and Twizzler
- Why pointers double from 64 to 128 bits — the extra 64 bits carry the bounds and permissions metadata that makes capabilities tamper-evident.
- Possibilities for CHERI + Twizzler
  
# CHERI Paper Writeup
# Brief Summary
CHERI swaps out  pointers for capabilities — hardware pieces that bundle a pointer together with its bounds/permissions. Instead of OS, the HW enforces it on every single access. So even if a pointer gets corrupted, it literally can't go out of bounds.
# Major Contributions

- Retrofits capabilities onto real ISAs (MIPS, RISC-V, Arm) — older systems like Multics needed totally new HW, CHERI doesn't.
- Monotonicity — you can only narrow a capability, never widen it. No way to forge a bigger one, the tag bit system makes sure of that.
- Works w/ C/C++  CHERI Clang  swaps pointers for capabilities at compile time.
- In-process compartmentalization — isolate components in the same address space w/o separate processes/page table switches.

# Strengths / Weaknesses
# Strengths:

- Incremental adoption -> don't have to rewrite everything at once
- Tag bits are a good solution with little hardware overhead
- Works even if the kernel is compromised since it's all hardware -> secure

# Weaknesses:

- Pointers go from 64 -> 128 bits, so more pressure + worse cache perf on pointer-heavy code
- Compressed bounds can't always represent exact byte ranges
- Still need to recompile w/ CHERI toolchain

# Questions
- Twizzler already does object-capabilities in SW — if you put CHERI under it, how much of that becomes redundant vs. actually additive?

# Reflections

I need to start annotating research papers as I read them rather than after.
I should write more consistently — both about what I'm actively working on and the things I'm stuck on or avoiding.
Talk to Daniel on Friday.
