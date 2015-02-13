
LeTest
======

Testing framework for Earl Grey.

Simple example:

    macros:
       require: letest -> tests
       {tests = tests}

    require:
       letest -> formatTests

    tests misc["Division by Zero"]:
       "when dividing a number by zero" =>
          "we get infinity" =>
             42 / 0 == Infinity
       "but when dividing zero by zero #bap" =>
          do:
             n = 0 / 0
          "we get a value which" =>
             "is not a number" =>
                String{n} == "NaN"
             "is not equal to itself" =>
                n != n

    formatTests{misc.select{}}

All tests are executed asynchronously and their results are collated
in the order that they finish. `do` blocks are executed before the
tests that follow them and can be used to set up state, but the tests
themselves may be executed in parallel and in no particular
order. Therefore, tests should not mutate data structures that may be
used by other tests, nor set up a state for other tests to use (all of
this can be done in the initial `do` block).

The `await` keyword can be used in the `do` blocks or in any test to
do asynchronous operations, e.g. fetch web pages.

The `inorder` keyword may be used to guarantee that the tests will be
executed in the order of their definition. For instance, the following
tests will only pass if the `inorder` directive is used. Otherwise
they will all fail because the second to fifth shift tests will be
executed while the first is waiting.

    wait = promisify{{d, f} -> setTimeout{f, d}}
    tests ordered:
       inorder "shift" =>
          do:
             x = {1, 2, 3, 4, 5}
          test:
             await wait{1000}
             x.shift{} == 1
          x.shift{} == 2
          x.shift{} == 3
          x.shift{} == 4
          x.shift{} == 5


