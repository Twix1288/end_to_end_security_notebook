# Week 10 Report — ripgrep on Twizzler

## ripgrep on Twizzler — It Finally Works
After ten weeks of talking to the GitHub AI about Twizzler and grinding through compatibility hell, ripgrep is properly running on Twizzler. Parallel search, regex matching, object-level search, it all works finally without crashing.

This is a rough write-up before proper documentation gets finished. The goal is to get down how it actually works, what it needs, and how to run it before any of that gets lost.

##Why We Needed It
The reason this was worth doing is that Twizzler's memory model is object-based rather than file-based, which means most traditional search tooling just doesn't translate. Getting ripgrep working means there's now a fast search tool that understands Twizzler's native object model and the traditional filesystem layer at the same time.

## How the Port Actually Works
Three crates needed to changes: crates/cli, crates/core, and crates/ignore. The regex engine, the matching logic, the gitignore parser was left alone. The idea was to change as little of the core search logic as possible, because that's the part that makes ripgrep worth using in the first place.

### Searching Twizzler Objects Directly
The biggest addition is a new flag: --object-id. Twizzler doesn't organize everything into a traditional filesystem hierarchy since data lives in memory objects with object IDs. The flag lets you pass one of those IDs directly to ripgrep.
The way it works is the object ID gets appended to the list of search paths. Twizzler can memory-map objects into the process address space, so ripgrep treats that mapped region the same way it treats a memory-mapped file on Linux or Windows. No special search codepath was needed because mmap is already how ripgrep prefers to read large files. The flag lives in crates/core/flags/defs.rs and maps into the LowArgs and HiArgs structs that carry configuration through the search pipeline.

### Parallelism
ripgrep's speed comes from running multiple search workers in parallel. The number of threads it spawns normally comes from std::thread::available_parallelism() — which isn't implemented yet for Twizzler's target triple.
Rather than hardcoding a thread count, we pull it straight from the OS. Inside crates/ignore/src/walk.rs, behind a #[cfg(target_os = "twizzler")] compile gate, we call twz_rt_get_sysinfo() from Twizzler's C runtime and read the available_parallelism field from the returned system_info struct. That gives an accurate core count so ripgrep can spawn the right number of workers and actually scale to the hardware.

### Directory Traversal
The ignore and walkdir crates were built around Unix inodes. They use inode numbers to detect symlink loops and avoid hitting the same directory twice. Twizzler doesn't expose inodes the way POSIX does, so the existing code would panic or silently fail trying to call fs::MetadataExt inode methods.
We wrote Twizzler-specific implementations of DirEntryRaw::from_entry_os and DirEntryRaw::from_path that skip the inode requirements entirely. Directory walking works fine for normal use. The tradeoff is hard-link loop detection is gone for now, which is covered under limitations below. Devin from the deep wiki 

### Terminal Detection
ripgrep checks whether stdout is a TTY to decide on colorized output and whether to use line buffering or block buffering. That check runs through is_terminal(), which uses ioctl on Unix and handle introspection on Windows. Neither exists on Twizzler right now.

Rather than let it panic, I stubbed it out in crates/cli/src/lib.rs and wtr.rs using a compile-time conditional that hardcodes is_terminal() to false for stdin, stdout, and stderr. It's not a clean solution but it's stable. Google gemini was a good help for this cause I didn't really understand the code for this part very well.

## What You Need to Run It
A Twizzler build environment with the Rust toolchain targeting twizzler, the Twizzler C runtime available for linking (that's what twz_rt_get_sysinfo() needs), and the patched ripgrep source with changes in crates/cli, crates/core, and crates/ignore. Build it like any other Rust project targeting Twizzler — no special flags beyond the target triple.

## Running It
* Standard directory search: rg "pattern" ./src
* Search a Twizzler object by ID: rg "ERROR" --object-id 1234abcd-5678-efgh
* Search while ignoring gitignore rules and hidden files: rg -uu "debug_trace" .
* Explicit stdin read if piped input hangs: cat file | rg "pattern" -
* Help page: rg -help

## Known Limitations
* No color by default. Because is_terminal() always returns false, ripgrep thinks it's writing to a file and drops color. Pass --color always whenever you want colored output.
* Block-buffered output. Same root cause — ripgrep defaults to block buffering when it thinks it's not on a TTY. Output can appear in chunks rather than line by line. It's not a correctness issue, it just feels off when using it interactively.
* No hard-link loop detection. Inode tracking got removed to make directory traversal work. If the Twizzler filesystem has recursive hard-links or bad symlink structures, ripgrep won't catch the loop. Unlikely in practice but worth knowing.

## What's Next
Proper documentation is getting written now. The main things: fix terminal detection once Twizzler gets ioctl or something equivalent, revisit inode loop detection using a Twizzler-native object identity approach, and make --object-id handle invalid or inaccessible object IDs better.
