---
title: "Scaling Up Unit Testing"
teaching: 10
exercises: 5
questions:
- "How can we make it easier to write lots of tests?"
- "How can we know how much of our code is being tested?"
objectives:
- "Use parameterisation to automatically run tests over a set of inputs"
- "Use code coverage to understand how much of our code is being tested using unit tests"
keypoints:
- "We can assign multiple inputs to tests using parametrisation."
- "It's important to understand the **coverage** of our tests across our code."
- "Writing unit tests takes time, so apply them where it makes the most sense."
---

## Introduction

We're starting to build up a number of tests that test the same function, but just with different parameters. However, continuing to write a new function for every single test case isn't likely to scale well as our development progresses. How can we make our job of writing tests more efficient? And importantly, as the number of tests increases, how can we determine how much of our code base is actually being tested?

## Parameterising Our Unit Tests

So far, we've been writing a single function for every new test we need. But when we simply want to use the same test code but with different data for another test, it would be great to be able to specify multiple sets of data to use with the same test code. Test **parameterisation** gives us this.

So instead of writing a separate function for each different test, we can **parameterise** the tests with multiple test inputs. For example, in `tests/test_models.py` let us rewrite the `test_daily_mean_zeros()` and `test_daily_mean_integers()` into a single test function:

~~~
@pytest.mark.parametrize(
    "test_data, test_index, test_columns, expected_data, expected_index, expected_columns",
    [
        (
            [ [0.0, 0.0], [0.0, 0.0], [0.0, 0.0] ],
            [ pd.to_datetime('2000-01-01 01:00'),
                pd.to_datetime('2000-01-01 02:00'),
                pd.to_datetime('2000-01-01 03:00') ],
            [ 'A', 'B' ],
            [ [0.0, 0.0] ],
            [ datetime.date(2000, 1, 1) ],
            [ 'A', 'B' ]
        ),
        (
            [ [1, 2], [3, 4], [5, 6] ],
            [ pd.to_datetime('2000-01-01 01:00'),
                pd.to_datetime('2000-01-01 02:00'),
                pd.to_datetime('2000-01-01 03:00') ],
            [ 'A', 'B' ],
            [ [3.0, 4.0] ],
            [ datetime.date(2000, 1, 1) ],
            [ 'A', 'B' ]
        ),
    ])
def test_daily_mean(test_data, test_index, test_columns, expected_data, expected_index, expected_columns):
    """Test mean function works for array of zeroes and positive integers."""
    from catchment.models import daily_mean
    pdt.assert_frame_equal(daily_mean(pd.DataFrame(data=test_data, index=test_index, columns=test_columns)),
                           pd.DataFrame(data=expected_data, index=expected_index, columns=expected_columns))
~~~
{: .language-python}

Here, we use Pytest's **mark** capability to add metadata to this specific test - in this case, marking that it's a parameterised test. `parameterize()` function is actually a [Python **decorator**](https://www.programiz.com/python-programming/decorator). A decorator, when applied to a function, adds some functionality to it when it is called, and here, what we want to do is specify multiple input and expected output test cases so the function is called over each of these inputs automatically when this test is called.

We specify these as arguments to the `parameterize()` decorator, firstly indicating the names of these arguments that will be passed to the function (`test`, `expected`), and secondly the actual arguments themselves that correspond to each of these names - the input data (the `test` argument), and the expected result (the `expected` argument). In this case, we are passing in two tests to `test_daily_mean()` which will be run sequentially.

So our first test will run `daily_mean()` on `[ [0.0, 0.0], [0.0, 0.0], [0.0, 0.0] ]` (our `test` argument), and check to see if it equals `[0.0, 0.0]` (our `expected` argument). Similarly, our second test will run `daily_mean()` with `[ [1, 2], [3, 4], [5, 6] ]` and check it produces `[3.0, 4.0]`.

The big plus here is that we don't need to write separate functions for each of the tests - our test code can remain compact 
and readable as we write more tests and adding more tests scales better as our code becomes more complex.

> ## Exercise: Write Parameterised Unit Tests
>
> Rewrite your test functions for `daily_max()` and `daily_min()` to be parameterised, adding in new test cases for each of them.
>
> > ## Solution
> > Test function for `daily_max()`
> > ~~~
> > ...
> > @pytest.mark.parametrize(
> >     "test_data, test_index, test_columns, expected_data, expected_index, expected_columns",
> >     [
> >         (
> >             [ [0.0, 0.0, 0.0], [0.0, 0.0, 0.0], [0.0, 0.0, 0.0] ],
> >             [ pd.to_datetime('2000-01-01 01:00'),
> >                 pd.to_datetime('2000-01-01 02:00'),
> >                 pd.to_datetime('2000-01-01 03:00') ],
> >             [ 'A', 'B', 'C' ],
> >             [ [0.0, 0.0, 0.0] ],
> >             [ datetime.date(2000, 1, 1) ],
> >             [ 'A', 'B', 'C' ]
> >         ),
> >         (
> >             [ [4, 2, 5], [1, 6, 2], [4, 1, 9] ],
> >             [ pd.to_datetime('2000-01-01 01:00'),
> >                 pd.to_datetime('2000-01-01 02:00'),
> >                 pd.to_datetime('2000-01-01 03:00') ],
> >             [ 'A', 'B', 'C' ],
> >             [ [4, 6, 9] ],
> >             [ datetime.date(2000, 1, 1) ],
> >             [ 'A', 'B', 'C' ]
> >         ),
> >         (
> >             [ [4, -2, 5], [1, -6, 2], [-4, -1, 9] ],
> >             [ pd.to_datetime('2000-01-01 01:00'),
> >                  pd.to_datetime('2000-01-01 02:00'),
> >                  pd.to_datetime('2000-01-01 03:00') ],
> >             [ 'A', 'B', 'C' ],
> >             [ [4, -1, 9] ],
> >             [ datetime.date(2000, 1, 1) ],
> >             [ 'A', 'B', 'C' ]
> >         ),
> >     ])
> > def test_daily_max(test_data, test_index, test_columns, expected_data, expected_index, expected_columns):
> >     """Test max function works for array of zeroes and positive integers."""
> >     from catchment.models import daily_max
> >     pdt.assert_frame_equal(daily_max(pd.DataFrame(data=test_data, index=test_index, columns=test_columns)),
> >                            pd.DataFrame(data=expected_data, index=expected_index, columns=expected_columns))
> > ...
> > ~~~
> > {: .language-python}
> > and for `daily_min()`
> > ~~~
> > ...
> > @pytest.mark.parametrize(
> >     "test_data, test_index, test_columns, expected_data, expected_index, expected_columns",
> >     [
> >         (
> >             [ [0.0, 0.0, 0.0], [0.0, 0.0, 0.0], [0.0, 0.0, 0.0] ],
> >             [ pd.to_datetime('2000-01-01 01:00'),
> >                 pd.to_datetime('2000-01-01 02:00'),
> >                 pd.to_datetime('2000-01-01 03:00') ],
> >             [ 'A', 'B', 'C' ],
> >             [ [0.0, 0.0, 0.0] ],
> >             [ datetime.date(2000, 1, 1) ],
> >             [ 'A', 'B', 'C' ]
> >         ),
> >         (
> >             [ [4, 2, 5], [1, 6, 2], [4, 1, 9] ],
> >             [ pd.to_datetime('2000-01-01 01:00'),
> >                 pd.to_datetime('2000-01-01 02:00'),
> >                 pd.to_datetime('2000-01-01 03:00') ],
> >             [ 'A', 'B', 'C' ],
> >             [ [1, 1, 2] ],
> >             [ datetime.date(2000, 1, 1) ],
> >             [ 'A', 'B', 'C' ]
> >         ),
> >         (
> >             [ [4, -2, 5], [1, -6, 2], [-4, -1, 9] ],
> >             [ pd.to_datetime('2000-01-01 01:00'),
> >                  pd.to_datetime('2000-01-01 02:00'),
> >                  pd.to_datetime('2000-01-01 03:00') ],
> >             [ 'A', 'B', 'C' ],
> >             [ [-4, -6, 2] ],
> >             [ datetime.date(2000, 1, 1) ],
> >             [ 'A', 'B', 'C' ]
> >         ),
> >     ])
> > def test_daily_min(test_data, test_index, test_columns, expected_data, expected_index, expected_columns):
> >     """Test min function works for array of zeroes and positive integers."""
> >     from catchment.models import daily_max
> >     pdt.assert_frame_equal(daily_max(pd.DataFrame(data=test_data, index=test_index, columns=test_columns)),
> >                            pd.DataFrame(data=expected_data, index=expected_index, columns=expected_columns))
> > ...
> > ~~~
> > {: .language-python}
> {: .solution}
>
{: .challenge}

Try them out!

Let's commit our revised `test_models.py` file and test cases to our `test-suite` branch (but don't push them to the remote repository just yet!):

~~~
$ git add tests/test_models.py
$ git commit -m "Add parameterisation mean, min, max test cases"
~~~
{: .language-bash}


## Code Coverage - How Much of Our Code is Tested?

Pytest can't think of test cases for us. We still have to decide what to test and how many tests to run. Our best guide here is economics: we want the tests that are most likely to give us useful information that we don't already have. For example, if `daily_mean(np.array([[2, 0], [4, 0]])))` works, there's probably not much point testing `daily_mean(np.array([[3, 0], [4, 0]])))`, since it's hard to think of a bug that would show up in one case but not in the other.

Now, we should try to choose tests that are as different from each other as possible, so that we force the code we're testing to execute in all the different ways it can - to ensure our tests have a high degree of **code coverage**.

A simple way to check the code coverage for a set of tests is to use `pytest` to tell us how many statements in our code are being tested. By installing a Python package to our virtual environment called `pytest-cov` that is used by Pytest and using that, we can find this out:

~~~
$ pip3 install pytest-cov
$ python -m pytest --cov=catchment.models tests/test_models.py
~~~
{: .language-bash}

So here, we specify the additional named argument `--cov` to `pytest` specifying the code to analyse for test coverage.

~~~
...
tests/test_models.py .........                                            [100%]

---------- coverage: platform darwin, python 3.9.4-final-0 -----------
Name                  Stmts   Miss  Cover
-----------------------------------------
catchment/models.py      19     10    47%
-----------------------------------------
TOTAL                    19     10    47%

============================== 9 passed in 0.26s ===============================
~~~
{: .output}

Here we can see that our tests are doing okay - 47% of statements in `catchment/models.py` have been executed. But which statements are not being tested? The additional argument `--cov-report term-missing` can tell us:

~~~
$ python -m pytest --cov=catchment.models --cov-report term-missing tests/test_models.py
~~~
{: .language-bash}

~~~
...
Name                  Stmts   Miss  Cover   Missing
---------------------------------------------------
catchment/models.py      19     10    47%   22-35, 44
---------------------------------------------------
TOTAL                    19     10    47%
...
~~~
{: .output}

So there's two groups of statements not being tested. The last is at line 44, which is in the `daily_total()` function, for which we could write tests in the same manner as we have
for `daily_max()` and `daily_min()`. The other group is at lines 22-35, which is within the `read_variable_from_csv()` function.
Here we should consider whether or not to write a test for this function, and, in general, any other functions that may not be tested. Of course, if there are hundreds or thousands of lines that are not covered it may not be feasible to write tests for them all. But we should prioritise the ones for which we write tests, considering how often they're used, how complex they are, and importantly, the extent to which they affect our program's results.

Again, we should also update our `requirements.txt` file with our latest package environment, which now also includes `pytest-cov`, and commit it:

~~~
$ pip3 freeze > requirements.txt
$ cat requirements.txt
~~~
{: .language-bash}

You'll notice `pytest-cov` and `coverage` have been added. Let's commit this file and push our new branch to GitHub:

~~~
$ git add requirements.txt
$ git commit -m "Add coverage support"
$ git push origin test-suite
~~~
{: .language-bash}

> ## What about Testing Against Indeterminate Output?
>
> What if your implementation depends on a degree of random behaviour? This can be desired within a number of applications, particularly in simulations (for example, molecular simulations) or other stochastic behavioural models of complex systems. So how can you test against such systems if the outputs are different when given the same inputs?
>
> One way is to *remove the randomness* during testing. For those portions of your code that use a language feature or library to generate a random number, you can instead produce a known sequence of numbers instead when testing, to make the results deterministic and hence easier to test against. You could encapsulate this different behaviour in separate functions, methods, or classes and call the appropriate one depending on whether you are testing or not. This is essentially a type of **mocking**, where you are creating a "mock" version that mimics some behaviour for the purposes of testing.
>
> Another way is to *control the randomness* during testing to provide results that are deterministic - the same each time. Implementations of randomness in computing languages, including Python, are actually never truly random - they are **pseudorandom**: the sequence of 'random' numbers are typically generated using a mathematical algorithm. A **seed** value is used to initialise an implementation's random number generator, and from that point, the sequence of numbers is actually deterministic. Many implementations just use the system time as the default seed, but you can set your own. By doing so, the generated sequence of numbers is the same, e.g. using Python's `random` library to randomly select a sample of ten numbers from a sequence between 0-99:
>
> ~~~
> import random
> 
> random.seed(1)
> print(random.sample(range(0, 100), 10))
> random.seed(1)
> print(random.sample(range(0, 100), 10))
> ~~~
> {: .language-python}
>
> Will produce:
>
> ~~~
> [17, 72, 97, 8, 32, 15, 63, 57, 60, 83]
> [17, 72, 97, 8, 32, 15, 63, 57, 60, 83]
> ~~~
> {: .output}
>
> So since your program's randomness is essentially eliminated, your tests can be written to test against the known output. The trick of course, is to ensure that the output being testing against is definitively correct!
>
> The other thing you can do while keeping the random behaviour, is to *test the output data against expected constraints* of that output. For example, if you know that all data should be within particular ranges, or within a particular statistical distribution type (e.g. normal distribution over time), you can test against that, conducting multiple test runs that take advantage of the randomness to fill the known "space" of expected results. Note that this isn't as precise or complete, and bear in mind this could mean you need to run *a lot* of tests which may take considerable time.
{: .callout}

## Test Driven Development

In the [previous episode](../21-automatically-testing-software/index.html#what-is-software-testing) we learnt how to create *unit tests* to make sure our code is behaving as we intended.
**Test Driven Development** (TDD) is an extension of this.
If we can define a set of tests for everything our code needs to do, then why not treat those tests as the specification.

When doing Test Driven Development, we write our tests first and only write enough code to make the tests pass.
We tend to do this at the level of individual features - define the feature, write the tests, write the code.
The main advantages are:

- It forces us to think about how our code will be used before we write it
- It prevents us from doing work that we don't need to do, e.g. "I might need this later..."

You may also see this process called **Red, Green, Refactor**: 'Red' for the failing tests, 'Green' for the code that makes them pass, then 'Refactor' (tidy up) the result.

For the challenges from here on, try to first convert the specification into a unit test, then try writing the code to pass the test.

## Limits to Testing

Like any other piece of experimental apparatus, a complex program requires a much higher investment in testing than a simple one. Putting it another way, a small script that is only going to be used once, to produce one figure, probably doesn't need separate testing: its output is either correct or not. A linear algebra library that will be used by thousands of people in twice that number of applications over the course of a decade, on the other hand, definitely does. The key is identify and prioritise against what will most affect the code's ability to generate accurate results.

It's also important to remember that unit testing cannot catch every bug in an application, no matter how many tests you write. To mitigate this manual testing is also important. Also remember to test using as much input data as you can, since very often code is developed and tested against the same small sets of data. Increasing the amount of data you test against - from numerous sources - gives you greater confidence that the results are correct.

Our software will inevitably increase in complexity as it develops. Using automated testing where appropriate can save us considerable time, especially in the long term, and allows others to verify against correct behaviour.

{% include links.md %}
