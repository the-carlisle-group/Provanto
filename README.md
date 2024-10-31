# Provanto
Provanto is a testing framework for Dyalog APL, including the fully automated testing of
`⎕WC` user interfaces.

It is included with Dado, and may be run with the user command `]Dado.Runtests`. Dado expects the tests
to be in the namespace `Tests` and the code to be in the namespace `Main`, unless otherwise specified.
Provanto is also distributed as a single namespace script for copy/pasting into a project, with no need for
Dado. And, as a Dado project itself, Provanto may be included as a dependency in the normal way. 

## The Tests Namespace
Provanto requires one or more **test suites** which are namespaces containing test functions.  
The `Tests` namespace may be a test suite itself, or a parent namespace contain multiple test suites.
The Test functions must be named with the prefix `Test`.
A test suite may contain application specific helper functions. However, there are four reserved words
(function/operator names): `Assert` and `Try` should not be used in a test suite,
and  `Startup` and `Teardown` have special meaning outlined below.

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

> Note that the naked guard technique is not a requirement. The requirement is that if the
> assertion is true, the test function continues. Thus the naked guard may be avoided by using a trad function
> for the test or by assigning the result of `Assert`.

A test function may include additional expressions as necessary. It is typical to use two lines
for each assertion, the first to compute a value, the subsequent one to test the value.
For real examples, see the [tests for the Text2Date project](https://github.com/the-carlisle-group/Text2Date/tree/master/APLSource/Tests).

## Test Result Codes
A test should explicitly return a `0` for *Passed*.
The framework provides a `1` for *Failed*, or a `2` for *Broken*.
A test may explicilty return a `3` for *Not Applicable*, or or `4` for *Disabled*.

## Running the Tests
The `Run` function takes a test suite or parent namespace containing multiple test suites, and executes the tests:

~~~
      {R}←[X [Y]] Run T [C] 
~~~

T is a test suite or namespace of test suites. C is an optional namespace of code
the provision of which triggers code coverage reporting.
X is an optional Boolean flag that if set to 1 stops
the `Run` function on failed or broken tests. Y is an optional integer from 0 to 3 that progressively suppresses session output. R is a namespace full of useful results for further processing or reporting.

The following is an example from the Text2Date project:

~~~
      #.Provanto.Main.Run #.Text2Date.(Tests Main)
Running suite: Tests
Passed:       #.Text2Date.Tests.TestCenturyWindow
Passed:       #.Text2Date.Tests.TestComplexNumbers
Passed:       #.Text2Date.Tests.TestErrorTrapping
Passed:       #.Text2Date.Tests.TestExcelFormats
Passed:       #.Text2Date.Tests.TestFalsePostives
Passed:       #.Text2Date.Tests.TestFixedFormats
Passed:       #.Text2Date.Tests.TestInferFormat
Passed:       #.Text2Date.Tests.TestLeadingVariableElement
Passed:       #.Text2Date.Tests.TestMultipleFormats
Passed:       #.Text2Date.Tests.TestNumericData
Passed:       #.Text2Date.Tests.TestPattern
Passed:       #.Text2Date.Tests.TestUndelimitedFixedFormats
Passed:       #.Text2Date.Tests.TestVariableFormats
──────────────────────────────────────────────
 Total  Passed  Failed  Broken  N/A  Disabled 
    13      13       0       0    0         0 
──────────────────────────────────────────────

Code coverage: 100%
~~~

If code coverage is activated, the test namespace is automatically added to
the code measured for coverage.

## Startup and Teardown
Each test suite may contain a `Startup` function and a `Teardown` function. These are automatically run before and after the tests in the suite.
There may also be a `Startup` and `Teardown` function in the parent namespace of a set of suites, which are run once for all of the suites. 

## Testing GUI Code
Provanto provides simple tools for the automated testing of GUIs built with `⎕WC`.
The `#.Provanto.SampleGUIApp` namepace contains a sample GUI application and associated tests.
Typically the `Startup` function will create the GUI and initialize the primary form for automated testing.
The Startup function in the sample GUI app is:

~~~
Startup←{
     _←1 ##.Main.Run 0
     _←#.Provanto.QWC.Init ##.Main.fmSampleApp
     0
 }
~~~

This creates the GUI and then intializes the primary form, preparing it for automated tests.
Once this is done, a typical test might look like:

~~~
TestAllowDigitsProperty←{
     s←GetAppState 0
     e←Get'Name'
     Assert~s.AllowDigits:
     Enter'Name' 'A12B345C':
     Assert e.Text≡'ABC':
     Enter'AllowDigits':
     Assert s.AllowDigits:
     Enter'Name' '12345':
     Assert e.Text≡'ABC12345':
     Enter'AllowDigits':
     Assert~s.AllowDigits:
     0
 }
~~~

The `GetAppState` function is a user-provided function that gives easy access to the state of the application
It might return a namepace, a class, or anything at all. The `Get` and `Enter` functions are injected into 
the test namepace by `QWC.Init` (just like `Assert` and `Try` are injected). `Get` retrieves objects from the
primary form using as short a path as possible. `Enter` takes some action on a GUI object. In the case
of the checkbox `AllowDigits`, it simply toggles it. In the case of the edit oject `Name`, it enters the provided
text. `Enter` will expect different arguments for different types of objects, and do different things depending 
on the type of object.

Events may be fired using the `Fire` function.
