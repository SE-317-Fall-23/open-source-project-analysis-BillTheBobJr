# Assignment Submission

## Project Selected: pytest

## I. Introduction
- Briefly introduce the purpose of this section, which is to provide a detailed description of two test cases in the selected open-source project.

## II. Test Case 1: test_fail_and_continue_with_stepwise
### A. Description
- The purpose of this test case was to make sure when the --stepwise argument was used with a test case, that once a test failed, all following tests would be skipped.
### B. Gherkin Syntax (if applicable)
- If you choose to use Gherkin syntax, write the Gherkin scenario for this test case.
### C. Test Steps
- Enumerate the sequence of steps or actions involved in the test case.
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
- The initial state is a number of tests that are defined at the start of that file (I will attached the code segement bellow), these tests are designed to pass and fail in alternating order.

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
- The test calls runpytest on the predefined tests with arguments "-v", "--strict-markers", "--stepwise", "--fail", and then again later with arguments "-v", "--strict-markers", "--stepwise".  This is so the first time it will include tests that will fail, and the second time all the tests will pass.
### G. Expected State
- Once the tests are run, it checks to make sure that after a test fails, all following tests are skip.  It does this by adding the "--fail" argument in the first run then runs it again without that argument.
- 
## III. Test Case 2: 
### A. Description
- Provide a concise overview of the purpose of the second test case.
### B. Gherkin Syntax (if applicable)
- If you choose to use Gherkin syntax, write the Gherkin scenario for this test case.
### C. Test Steps
- Enumerate the sequence of steps or actions involved in this test case.
### D. Code Segments Under Test
- Identify specific code segments or functions that are being tested in this case.
```Java
```
### E. Initial State
- Describe the initial state or conditions of the system or component before executing the test.
### F. Transition
- Explain the actions or events that trigger a change in the system's state.
### G. Expected State
- Clearly outline the expected state or outcome after the test steps have been completed.

