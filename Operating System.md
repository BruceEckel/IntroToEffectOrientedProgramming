"Operating Environment"
"Concurrency" == async and parallel

Early programmers wrote directly to the hardware. 
Each program had to understand how to manipulate the particular hardware it ran on, and this was true even on large, expensive machines.
This generated large amounts of duplicated code and required rewrites to run the software on different machines.

Operating systems provided a common interface to the underlying hardware.
All that custom, duplicated code coalesced into a layer maintained separately from user programs.
This dramatically increased programming value by eliminating all the specialized coding and debugging necessary to talk to the hardware.
It also created the possibility to write portable programs, at least within the same operating system across different hardware.

Although the operating system was a great step forward, from a higher perspective it is still a rather primitive step.
The programmer must still think about many low-level details.
It's a slightly higher abstraction, but we must still write, debug, and maintain too much code for performing the same tasks over and over again for each new program.
In addition, these systems are brittle--making changes can require extensive re-coding or even re-design.

What if we dramatically raise the abstraction of the operating system?
Arguably, the standard library for your programming language does this to some degree.
You use a "print" function from your standard library to display something on the console--you don't need to know the operating system call.
Even with a large standard library, however, these are still relatively basic operations.
It's a better abstraction, but it doesn't liberate us from repetitive code and system fragility.
On top of that, the problems of concurrency (both async and parallel execution) must be solved in a custom manner for each new program.

We've come to accept these things as "what programming is."

Consider a much higher-level operating-system-like entity that eliminates these code-duplication and fragility problems.
It manages I/O, concurrency, and higher-level issues that allow you to design *what the program does* rather than *how it does it*.
Modifying a program in such a system becomes far easier and faster.
For now, let's call this an "operating environment."

For example, suppose you must connect with another system to fetch or deliver information.
The remote system turns out to be unreliable in various ways: it doesn't always respond right away, and sometimes not at all.
You must try various approaches to deal with this unreliability.
Each approach typically requires significant amounts of consideration and coding, and you don't know ahead of time which ones will work.

Imagine each of these operations is encapsulated in such a way that those operations can be easily composed.
Communicating with another system is an encapsulated operation.
Fallbacks and retries and other approaches are also encapsulated operations.

To compose, you stick them together, and the "operating environment" takes care of details that would ordinarily take days, weeks, or months to set up and maintain.
If an experiment doesn't produce the desired result, it can be easily changed to try another approach.
If there's an opportunity to use concurrency, the "operating environment" does it automatically.
This raises the level of abstraction far closer to "what" and further away from "how."

## The Price

The benefits of such a system are obvious: programmers spend far more of their time creating good programs and far less time getting those programs to work.
Fewer programmers build and maintain much more sophisticated systems.
Customers get much more for much less.

With such amazing business value, why isn't everyone creating systems this way?
There is a cost: the significant change in the mental model of programming.
Most programmers are not trained or experienced in this kind of mental shift or in thinking at this level of abstraction.
Our goal in this book is to simplify these ideas to make them more easily adoptable.

Programmers are understandably suspicious of _the next great thing_.
We have all been taken in by promises that have not panned out.
Perhaps the biggest has been object-oriented programming (OOP).
OOP has probably had such staying power because it had types (a great thing) at its core; unfortunately, it also saddled us with inheritance.

Although the benefits are tremendous, the cost is significant, which means this revolution will take some time.
It's worth considering the costs before jumping into the mix.

## The Model

To achieve this magical behavior, we must change program creation:

1. Each operation keeps track of anything it does that touches the environment. An operation must communicate all details of environmental effects to other operations, so the "operating environment" can coordinate that operation with other operations as well as manage concurrency.
2. Operations must be represented as "runnable elements." This gives the "operating environment" the flexibility to decide when and how to execute those operations.
