Early programmers wrote directly to the hardware. 
Each program had to understand how to manipulate the particular hardware it ran on, and this was true even on large, expensive machines.
This generated large amounts of duplicated code and required rewrites in order to run the software on different machines.

Operating systems provided a common interface to the underlying hardware.
All that custom, duplicated code coalesced into a layer maintained separately from user programs.
This dramatically increased programming value by eliminating all the specialized coding and debugging necessary to talk to the hardware.
It also created the possibility to write portable programs, at least within the same operating system across different hardware.

Although the operating system was a great step forward, from a higher perspective it is still a rather primitive step.
The programmer must still think about many low-level details.
It's a slightly higher abstraction, but we must still write, debug and maintain too much code for performing the same tasks over and over again for each new program.
In addition, these systems are brittle--making changes can require extensive re-coding or even re-design.

What if we dramatically raise the abstraction of the operating system?
Arguably, the standard library for your programming language does this to some degree.
To display something on the console, you don't need to know the operating system call, but instead use a "print" function from your standard library.
