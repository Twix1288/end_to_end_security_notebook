# Week 3

# What I Did
This week, I dove deep into the weeds of terminal and file descriptor handling to get ripgrep playing nicely with Twizzler. Before making any Rust runtime changes, I spent a lot of time reading over Daniel's branch, specifically his updated mlibc implementations. I wanted to see exactly how syscalls were behaving at the C library layer before they even hit the Rust ABI.

By tracing through his sysdeps.cpp, I found that sys_isatty(int fd) unconditionally returns 0, and sys_ioctl defaults to returning ENOSYS. This was a huge lightbulb moment for understanding why ripgrep's terminal UI was struggling.

# Troubles
I discovered a nasty coupling problem: isatty() and ioctl(TIOCGWINSZ) (which queries terminal window size) are deeply intertwined. If I were to patch isatty() to return true, ripgrep's termcolor crate would immediately try to run ioctl, hit that ENOSYS error, and either crash or spew ANSI garbage.

Furthermore, isatty() currently can't distinguish between a true kernel console and a redirected file (e.g., rg pattern > out.txt) because Twizzler's FdKind::Stdio variant isn't fully aware of redirections yet.

# Goals and Aspirations
My goal for now is to make the pragmatic choice: keep isatty() returning false so we don't break file redirections. However, I plan to add a safety-net ioctl(TIOCGWINSZ) stub in mlibc that returns a default 80x24 terminal size and an ENOTTY error for unknown requests. This lays the groundwork for eventually enabling true terminal support.
