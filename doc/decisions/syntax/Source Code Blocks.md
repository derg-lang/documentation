# Overview

Virtually all programming languages utilizes blocks of code to encapsulate some
concept, behavior, or state. There exists multiple mechanisms for how such
blocks could be defined. Many familiar languages utilizes specific tokens to
indicate the start and end of such blocks (i.e. C with `{` and `}`, Fortran
with `then` and `end`, etc). Other languages utilize whitespace and indentation
alone to define the blocks (i.e. Python).

Derg is a programming language which relies on explicitly defining where a block
starts and ends, rather than relying on indentation alone. All explicit code
blocks open with `{` and close with `}`.

## Semantically meaningful whitespace

While whitespace-only languages such as Python has found success, there are
certain qualities with using semantic whitespace which are known to cause
trouble. While fairly easy to combat, utilizing semantic whitespace immediately
poses the question of how to handle different types of whitespace characters,
such as spaces vs. tabs and other more exotic forms.

Working with code utilizing whitespace can cause unexpected issues when moving
bulks of code around. Consider a function containing deeply nested statements.
If the function is moved to another location, or the contents refactored to
changing to a different indentation level, then the tooling must automatically
handle the change in whitespace after pasting. If this routine is incorrectly
implemented, the source code may change its semantics upon pasting code.

## Token-denoted code blocks

Utilizing token-based denoting of blocks does not have the same downside. Moving
code denoted by blocks may have broken indentation after pasting, but its
semantics remains unchanged. Ensuring the code remains the same before and after
such transformation is a valued property.

Token-based approach has the downside of additional visual clutter in the form
of tokens, however. Every block is required to start with an open token, and end
with a close token (`{` and `}`, respectively). At present, no research
examining readability impacts of curly brackets is known, and as such cannot be
used to make an informed decision of such impact.
