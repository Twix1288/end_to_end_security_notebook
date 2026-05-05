# Week 3

# What I Did
This week, I focused on foundational research to figure out how mlibc, syscalls, and the general porting process might work. I used Devin to help trace through the architecture and get a better grasp of the workflow. Since I am planning to port a new tool, getting this baseline understanding of how C standard libraries interface with the OS was crucial.

# Troubles
I ran into a lot of problems navigating the codebase and trying to map everything out. It required going over multiple parts of the system to figure out exactly what was already implemented and what was missing. It's a bit of a maze trying to track down unimplemented features and syscalls from scratch.

## Goals and Aspirations
My goal is to pinpoint exactly what is missing in the toolchain and mlibc so I can start making concrete steps toward the porting process without shooting in the dark.
