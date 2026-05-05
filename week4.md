# Week 4

# What I Did
I emailed Daniel this week, and he told me about a specific branch he’s been actively updating, which gave me a much better starting point for my work. I also had a really helpful conversation with Sean; he pointed me in the right direction for finding the syscall and sysinfo information I had been struggling with during Week 1.

I spent some time evaluating the toolchain to see if there was anything I needed that wasn't currently there. Furthermore, I dug deeper into ripgrep and realized that porting it is a very CPU-intensive process. I spent a good chunk of time figuring out how to ensure ripgrep has a possibility of working natively with Twizzler's object IDs.

# Troubles
Mapping standard file I/O to an object ID system is proving to be a complex hurdle. Also, tracking down exactly what is missing in the toolchain and ensuring my environment matches up with the changes in Daniel's branch is an ongoing balancing act.

# Goals and Aspirations
Get a solid grasp on the object ID implementation for ripgrep and make sure my local toolchain is fully synced up with the latest necessary branches so I can start compiling.
