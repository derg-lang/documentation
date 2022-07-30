# Introduction

In the current modern times, software becomes more and more prevalent and
mandatory, in every industry. As software requirements grow, so too, does the
need for secure, robust, maintainable, and scalable solutions.

Today multiple large organizations are left with massive code bases of legacy
code, typically written for another era where the programming paradigms and ways
of thinking were different. Many of these code bases cannot be re-written into a
different programming language and often doing so would make no sense - the old
code works and is battle-hardened.

However, these organizations are left with a painful choice - keep working on
legacy software using the existing programming language, or re-write potentially
large swaths of business logic in a different tech stack, and somehow integrate
the two worlds with each other.

Maintaining legacy software is a painful endeavor and due to outdated
development practices and models, the programming languages of older times are
not ideal for writing robust and secure software.

Furthermore, as the world of software evolves onwards, becoming ever-more
virtualized, multiple smaller services are developed and deployed by
organizations. These smaller services are frequently written in various
programming languages, and maintained by different teams within each
organization.

All such services may require the same shared functionality, but are unable to
efficiently share code. While solutions exists, such as sharing functionality by
developing yet another service, there are many performance gains and
architectural improvements to be found by sharing *source code*.

# The guiding principles of Derg

Derg seeks to binding together all technologies and programming languages by
providing a unified language to serve all use-cases. Any code written in Derg,
will be immediately available in every other supported programming language as
well. All semantics and behavior of the Derg program will continue to function
exactly as desired, as if the target language was Derg itself.

Simply fulfilling a role is not enough for a programming language, however. Any
programming language must fulfill the needs of the present, and the future. As
such, Derg must be a language which is maintainable, scalable, robust, and
otherwise capable of dealing with all hardship software development imposes on
it.

Thus, there are some properties Derg must prioritize over others, which will
form the core principles influencing all future design choices. In every choice
made in the design of Derg, there are tradeoffs to be considered. Not every
decision will "be the right choice" for every developer in the world, although
every decision must follow the guiding principles of Derg.

## Prefer simplicity to explicitness

During the lifetime of software, code must be written, tested, and maintained.
While the code is typically written once and changed multiple times afterwards,
it is read many, many times over.

Source code is not examined only to find a bug or flaw, but also to determine
how best to integrate new features, or to understand how the software works. By
optimizing for ease of reading, the maintenance burden will be somewhat lessened
at the cost of requiring more effort to write the code initially.

Any programming language has grammar which describes how source code is
structure and should be parsed. In many programming languages, large parts of
the syntax is mandatory and can never be omitted, or otherwise force large
swaths of boilerplate to be written by any developer.

When any element in Derg source code is optional and introduces unnecessary
visual clutter without notably improving comprehension or readability, such
elements should be removed. This ensures any code written remains as sleek and
clean as it possibly can be.

## Prefer reliability to performance

Modern software must be reliable, and must be trusted to do the right thing at
every critical moment. While performance is always to strive for, Derg
recognizes that functioning software is crucial for serving the requirements of
the future world.

Security becomes an ever more important aspect as time moves on, and as such
writing safe software is crucial. By default, in matters where undefined
behavior can be introduced to gain performance, reliability trumps gains. Safe
code should not cause segmentation faults, out-of-bounds accesses, or concurrent
modification of memory.

As a side-note, performant data structures becomes an opt-in rather than
opt-out. All data structures and algorithms in the standard library would be
default be safe, whereas specialized data structures and algorithms may be
provided where the user desires performance over everything else.
