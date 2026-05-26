# **Week 7 Update: ripgrep on Twizzler**

ripgrep works on Twizzler now. Here's what I did to make that happen.

## **Codebase Integration & Build System**

The ripgrep source tree got vendored into the Twizzler repo as a workspace member under `src/ports/ripgrep`. The initrd build pipeline was updated to include the `rg` binary so it's available on boot.

There was a panic in the `xtask` build system where a `.canonicalize().unwrap()` call blew up when `clang` wasn't found. Changed it to `.unwrap_or(clang_path)` so it fails gracefully instead.

## **Ripgrep Twizzler-Specific Patches**

`cfg(unix)` is false on Twizzler, which broke a few things in ripgrep's source that assume a UNIX environment.

Directory traversal had to be patched because `DirEntryRaw` relies on inode numbers and `std::os::unix` APIs that don't exist on Twizzler. Added a `target_os="twizzler"` implementation that works around that. `std::thread::available_parallelism()` also isn't implemented yet, so thread pool sizing was patched to use `twz_rt_get_sysinfo()` instead. The `is_terminal()` and hyperlink checks also needed small patches to not crash on a non-UNIX target.

## **Twizzler Runtime Support**

ripgrep needs libc-style environment variable functions that the Twizzler runtime wasn't providing. Daniel helped me figure out where to add them.

`getenv` was rewritten to cache C-string results in a `Mutex<BTreeMap<String, CString>>` so the pointer stays valid between calls, which is what POSIX requires. `setenv` and `unsetenv` were added as weakly-linked C-ABI shims that call into Rust's `std::env::set_var` and `std::env::remove_var`. `set_var` is deprecated in Rust 1.81+ for being unsound in multithreaded contexts, but all env mutations happen before any threads are spawned here so it's fine. Left a comment explaining that rather than just suppressing the warning.

## **Init Shell Improvements**

The shell needed double-quote parsing so something like `rg "fn main" /initrd/` doesn't get split into three arguments. Added `shell_split()` for that. Also added `PWD=/` injection into spawned compartments so `getcwd()` doesn't fail, and refactored `as_env` to use owned strings to fix some lifetime issues.

## **Troubles**

The problems compounded on each other a lot. Fixing one thing would expose the next missing piece. The `getenv` rewrite took longer than it should have. Sean and Daniel both helped me get unstuck at various points.

## **Goals**

I still need to write documentation for the runtime additions and the general porting process. Also want to clean up some shell stuff. ripgrep works though, so that's week 7.
