# Assignment Submission

## Project Selected: pytest

## I. Introduction
- The two tests I selected are in the test_stepwise package.  This test are specifcally for the --stepwise argument.  Using this argument will make it so you only run tests until you fail one, at which point you skip the rest of the tests.  Here I will go more indepth on two of the tests found in this package.

## II. Test Case 1: test_fail_and_continue_with_stepwise
### A. Description
- The purpose of this test case was to make sure when the --stepwise argument was used with a test case, that once a test failed, all following tests would be skipped.
### B. Gherkin Syntax (if applicable)
- I chose not to use Gherkin Syntax
### C. Test Steps
- We run the predefined tests, forcing the second test to fail
- We analyze the tests, making sure no warnings/exceptions occur and only the first 2 tests execute and the third does not, where test 1 passes and test 2 fails
- We run the predefined tests again, this time making test 2 pass
- We analyze the tests, making sure no warnings/exceptions occur and the second and third test ran, both of which pass
### D. Code Segments Under Test
- The runpytest function is what is being tested, specifically, when the --stepwise argument is used

```Python
def test_fail_and_continue_with_stepwise(stepwise_pytester: Pytester) -> None:
    # Run the tests with a failing second test.
    result = stepwise_pytester.runpytest(
        "-v", "--strict-markers", "--stepwise", "--fail"
    )
    assert _strip_resource_warnings(result.stderr.lines) == []

    stdout = result.stdout.str()
    # Make sure we stop after first failing test.
    assert "test_success_before_fail PASSED" in stdout
    assert "test_fail_on_flag FAILED" in stdout
    assert "test_success_after_fail" not in stdout

    # "Fix" the test that failed in the last run and run it again.
    result = stepwise_pytester.runpytest("-v", "--strict-markers", "--stepwise")
    assert _strip_resource_warnings(result.stderr.lines) == []

    stdout = result.stdout.str()
    # Make sure the latest failing test runs and then continues.
    assert "test_success_before_fail" not in stdout
    assert "test_fail_on_flag PASSED" in stdout
    assert "test_success_after_fail PASSED" in stdout
```
### E. Initial State
- The initial state is a number of tests that are defined at the start of that file (I will attached the code segement bellow), these tests are designed to pass and fail in alternating order so long as the --fail argument is passed.

```Python
def stepwise_pytester(pytester: Pytester) -> Pytester:
    # Rather than having to modify our testfile between tests, we introduce
    # a flag for whether or not the second test should fail.
    pytester.makeconftest(
        """
def pytest_addoption(parser):
    group = parser.getgroup('general')
    group.addoption('--fail', action='store_true', dest='fail')
    group.addoption('--fail-last', action='store_true', dest='fail_last')
"""
    )

    # Create a simple test suite.
    pytester.makepyfile(
        test_a="""
def test_success_before_fail():
    assert 1

def test_fail_on_flag(request):
    assert not request.config.getvalue('fail')

def test_success_after_fail():
    assert 1

def test_fail_last_on_flag(request):
    assert not request.config.getvalue('fail_last')

def test_success_after_last_fail():
    assert 1
"""
    )

    pytester.makepyfile(
        test_b="""
def test_success():
    assert 1
"""
    )

    # customize cache directory so we don't use the tox's cache directory, which makes tests in this module flaky
    pytester.makeini(
        """
        [pytest]
        cache_dir = .cache
    """
    )

    return pytester
```

### F. Transition
- The test calls runpytest on the predefined tests with arguments "-v", "--strict-markers", "--stepwise", "--fail", and then again later with arguments "-v", "--strict-markers", "--stepwise".  This is so the first time it will force some tests to fail, and the second time all the tests will pass.
### G. Expected State
- Once the tests are run, it checks to make sure that after a test fails, all following tests are skip.  It forces some tests to fail with the --fail argument and makes all tests after a fail skip with the --stepwise argument.


## III. Test Case 2: test_change_testfile
### A. Description
- This test makes sure if you run a stepwise test on one file, then run a different stepwise test on a different file, that it will run properly
### B. Gherkin Syntax (if applicable)
- I chose not to use Gherkin Syntax.
### C. Test Steps
- We run test_a test file using stepwise and making sure some tests fail
- We analyze the output to ensure that we fail and after that we skip the rest of the tests
- We then run test_b test file stepwise
- We analyze the output to ensure that it runs properly, and that running stepwise for some other file doesn't effect this files output
### D. Code Segments Under Test
```Python
def test_change_testfile(stepwise_pytester: Pytester) -> None:
    result = stepwise_pytester.runpytest(
        "-v", "--strict-markers", "--stepwise", "--fail", "test_a.py"
    )
    assert _strip_resource_warnings(result.stderr.lines) == []

    stdout = result.stdout.str()
    assert "test_fail_on_flag FAILED" in stdout

    # Make sure the second test run starts from the beginning, since the
    # test to continue from does not exist in testfile_b.
    result = stepwise_pytester.runpytest(
        "-v", "--strict-markers", "--stepwise", "test_b.py"
    )
    assert _strip_resource_warnings(result.stderr.lines) == []

    stdout = result.stdout.str()
    assert "test_success PASSED" in stdout
```
### E. Initial State
- The initial state is a number of tests that are defined at the start of that file (I will attached the code segement bellow), these tests are designed to pass and fail in alternating order so long as the --fail argument is passed.  Specifially note how there are 2 different files defined in the method to create our input, this is so we can switch between them in one test. These test files are test_a and test_b.

```Python
def stepwise_pytester(pytester: Pytester) -> Pytester:
    # Rather than having to modify our testfile between tests, we introduce
    # a flag for whether or not the second test should fail.
    pytester.makeconftest(
        """
def pytest_addoption(parser):
    group = parser.getgroup('general')
    group.addoption('--fail', action='store_true', dest='fail')
    group.addoption('--fail-last', action='store_true', dest='fail_last')
"""
    )

    # Create a simple test suite.
    pytester.makepyfile(
        test_a="""
def test_success_before_fail():
    assert 1

def test_fail_on_flag(request):
    assert not request.config.getvalue('fail')

def test_success_after_fail():
    assert 1

def test_fail_last_on_flag(request):
    assert not request.config.getvalue('fail_last')

def test_success_after_last_fail():
    assert 1
"""
    )

    pytester.makepyfile(
        test_b="""
def test_success():
    assert 1
"""
    )

    # customize cache directory so we don't use the tox's cache directory, which makes tests in this module flaky
    pytester.makeini(
        """
        [pytest]
        cache_dir = .cache
    """
    )

    return pytester
```

### F. Transition
- The test calls runpytest specifically passing in the arguments "-v", "--strict-markers", "--stepwise", "--fail", "test_a.py".  This is to make the tests run in stepwise, make some tests fail, and to ensure we are running the tests defined in test_a.  We later sun runpytest again but passing in "-v", "--strict-markers", "--stepwise", "test_b.py" arguments. This is to make sure we use the tests in the test_b file defined above.
### G. Expected State
- Finally we use assets to ensure that both runs of runpytest act properly.

