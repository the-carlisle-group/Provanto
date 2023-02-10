# Provanto
Provanto is a testing framework for Dyalog APL.

It is included with Dado, and may be run with the user command `]dada.Runtests`. Dado expects the code to be in 
`Main` and the tests to be in `Tests`, unless otherwise specified.  

Provanto is also distributed as a single namespace script for easy copy/pasting into a project, with no need for
Dado.

And, as a Dado project itself, Provanto may be included as a dependency in the normal way. 

## The Tests Namespace
Provanto requires a namespace containing test functions. Test functions must be named with the prefix `Test`.
The Tests namespace may contain application specific helper functions. However, there are two reserved words
(function/operator names): `Assert` and `Try`. These names should not be used in the Tests namespace. 

A test function is a gauntlet of assertions terminated by a `0`. For example:

