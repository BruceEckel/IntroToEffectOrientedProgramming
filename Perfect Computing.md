# Perfect Computing

Suppose you could build the perfect computing platform.
To achieve this, we need to be able to produce perfect composition.
That means I can take any component and stick it into your system and it will play well with the other components.
To achieve that, a component needs to be perfectly opaque.
Each component only exposes its interface, using the same universal description mechanism that all components use.

Composability is one of the fundamental holy grails of computing.

Now think about what it would take to achieve this opaqueness.
You can know nothing--*nothing*--about what's inside a component.
You cannot know the computing model or the programming language used to implement that component.
There can be no leaks.

You can't know the programming language or the architecture of the implementation.
So what happens if there's a failure in that component?
If I'm programming in C, I might return an unlikely value and hope the caller notices.
But if I'm coding in Java or Python, I'll throw an exception.

What do you, a perfectly opaque component, do with an exception?
Throwing exceptions is *not* playing well with others.
Leaving a mess or behaving unpredictably is *not* playing well with others.
Playing well with others is taking a request and providing a result, and nothing else.

*Pure functions* take a request and provide a result, and nothing else.
Because of this, pure functions are easily composed.

*Effectful Functions* may or may not provide a result, but they do other things.
Those other things impose "side effects" on the rest of the world, or are affected by the world.
The state of the world changes.
The function behaves unpredictably.
It leaves a mess.
It does not compose.
It breaks our perfect computing platform.

In most programming languages today, programmers must keep track of the messes.
Programmers spend far too much time managing messes to even hope for a system that does it for them.

Rather than manage the messes, we want to keep them from happening in the first place.
We need to pay attention to the things that are making the messes.
We need to plug the leaks.

What are these leaks?
Exceptions are an obvious leak because they just blow everything up and show up later ... somewhere.
Hoping that someone pays attention to it, like a drunken partygoer after an all-nighter.
Any sort of IO, from console to network, leaks its bytes out into the world, and leaves a mess.

Perhaps surprisingly, *reading* from the world is also a side effect, because that information changes the result produced, which in turn changes the rest of the system.
An impure function is unpredictable; because of the impurity you can call it multiple times with the same inputs and not get the same results.

A system of pure functions is pointless: it never touches the world, so it cannot influence the world, except to warm it slightly.
Impurity is essential, but our problem isn't impurity, it's ignoring side effects.
To pay attention to side effects, we must first know about them.
We need a way to package the side-effect information so we can tell component users, and other components, about it.
If everything that touches the component knows about the side effects, then that information doesn't get lost.
With that information, we can build a system that doesn't leave messes.
A system that composes.

Rather than requiring you to run around and plug leaks as they spring up, an *Effect System* plugs the leaks for you.
Doing so allows it, and by extension you, to do more, much more easily:
[[List from superpowers]]

All this and:

- You don't lose errors
- You can create deterministic tests

There's also the initialization problem: a program needs resources, which often rely on each other.
When programmers figure out those dependencies, they can't know for sure if they got it right.
If anything changes they have to figure them out again.
Because it keeps track of effects, Effect Systems solve that problem, provably.

- We're not getting enough information from the function.
  Put that information in the return value.
  Types are essential here, because they manage the information in that return value.
