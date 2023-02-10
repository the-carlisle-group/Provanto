# Provanto
Provanto is a testing framework for Dyalog APL.

It is included with Dado, and may be run with the user command `]Dado.Runtests`. Dado expects the code to be in 
`Main` and the tests to be in `Tests`, unless otherwise specified.  

Provanto is also distributed as a single namespace script for copy/pasting into a project, with no need for
Dado.

And, as a Dado project itself, Provanto may be included as a dependency in the normal way. 

## The Tests Namespace
Provanto requires a namespace containing test functions. Test functions must be named with the prefix `Test`.
The Tests namespace may contain application specific helper functions. However, there are two reserved words
(function/operator names): `Assert` and `Try`. These names should not be used in the Tests namespace. 

A test function is a gauntlet of assertions terminated by a `0`. For example, to test the primitive
function `+` we might write:

~~~
TestPrimitivePlus←{
     Assert 4=2+2:
     Assert 2 3 4≡1 2 3+1:
     Assert 5 6≡2+3 4:
     Assert 5=2 3+Try 4 5 6:
     Assert 11=2+Try'A':
     0
 }
~~~

The `Assert` function requires a scalar Boolean right argument. It preceeds a naked guard. 
The `Try` operator attempts to apply
the function with the given arguments, and returns the error number or 0 if no error is generated.

A test function may include additional expressions as necessary. It is typical to use two lines
for each assertion, the first to compute a value, the subsequent one to test the value.
For real examples, see the [tests for the Text2Date project](https://github.com/the-carlisle-group/Text2Date/tree/master/APLSource/Tests).

## Test Result Codes
A test should explicilty return a `0` for Passed.
The framework provides a `1` for Failed, or a `2` for Broken.
A test may explicilty return a `3` for Not Applicable, or or `4` for Disabled.

## Running the Tests
The `Run` function takes a test namespace and executes the tests.
The following is an example from the Text2Date project:

~~~
      #.Provanto.Run #.Text2Date.Tests
Passed:       TestCenturyWindow
Passed:       TestFixedFormats
Passed:       TestLeadingVariableElement
Passed:       TestPattern
********************
 Number of tests:  4
 Passed            4
 Failed            0
 Broken            0
 N/A               0
 Disabled          0
********************
Code coverage:  n/a
~~~

The code coverage feature is activated by providing the parent code namespace as an optional argument:

~~~
      #.Provanto.Run #.Text2Date.(Tests Main)
Passed:       TestCenturyWindow
Passed:       TestFixedFormats
Passed:       TestLeadingVariableElement
Passed:       TestPattern
********************
 Number of tests:  4
 Passed            4
 Failed            0
 Broken            0
 N/A               0
 Disabled          0
********************
Code coverage:  78%
Untested code:
#.Text2Date.Main.BasicFormats            
#.Text2Date.Main.InferFormat             
#.Text2Date.Main.Signal                  
#.Text2Date.Main.VariableMMM[3 4 5 6 7 8]
~~~

The untested functions, or specfic untested lines, are displayed.

The optional left argument to `Run` provides options to stop on a failing test, stop on a broken test,
and to supress session output.
