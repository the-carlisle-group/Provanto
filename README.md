# Provanto
Provanto is a testing framework for Dyalog APL.

It is included with Dado, and may be run with the user command `]Dado.Runtests`. Dado expects the code to be in 
`Main` and the tests to be in `Tests`, unless otherwise specified.  

Provanto is also distributed as a single namespace script for copy/pasting into a project, with no need for
Dado.

And, as a Dado project itself, Provanto may be included as a dependency in the normal way. 

## The Tests Namespace
Provanto requires one or more **test suites** which are namespaces containing test functions.  
The `Tests` namespace may be a test suite itself, or a parent namespace contain multiple test suites.
The Test functions must be named with the prefix `Test`.
A test suite namepace may contain application specific helper functions. However, there are four reserved words
(function/operator names): `Assert` and `Try` should not be used in in the Tests namespace, and  `Startup` and `Teardown`
have special meaning outlined below.

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
The `Run` function takes a test suite or parent namespace containing multiple test suites, and executes the tests.
The following is an example from the Text2Date project:

~~~
      #.Provanto.Run #.Text2Date.Tests
Passed:       #.Text2Date.Tests.TestCenturyWindow
Passed:       #.Text2Date.Tests.TestFixedFormats
Passed:       #.Text2Date.Tests.TestLeadingVariableElement
Passed:       #.Text2Date.Tests.TestPattern
********************
 Number of tests:  4
 Passed            4
 Failed            0
 Broken            0
 N/A               0
 Disabled          0
********************
~~~

A code coverage feature is activated by providing the parent code namespace as an optional argument:

~~~
      #.Provanto.Run #.Text2Date.(Tests Main)
Passed:       #.Text2Date.TestCenturyWindow
Passed:       #.Text2Date.TestFixedFormats
Passed:       #.Text2Date.TestLeadingVariableElement
Passed:       #.Text2Date.TestPattern
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

If code coverage is activated, the test namespace is automatically added to
the code measured for coverage.

The optional left argument to `Run` provides options to stop on failing and broken tests,
and to suppress session output.

## Startup and Teardown
Each test suite may contain a `Startup` function and a `Teardown` function. These are automatically run before and after the tests in the suite.
There may also be a `Startup` and `Teardown` function in the parent namespace of a set of suites, which are run once for all of the suites. 

