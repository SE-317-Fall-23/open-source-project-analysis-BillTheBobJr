## Project Selected: pytest

## I. Introduction
- Here I will do an analysis of the testing for this project as a whole, inparticular, looking at what type of data generation is used for the testing.

## II. Test Data Generation
### A. Static Test Data
- Static Test Data is used in this project. This is because a lot of the tests in this projec, require running a file and confirming the output as expected. Because the test values are formated as files a lot of times, it makes it difficult to randomly generate values to test.  So they will often just create files that have cater to the cases they wise to test.
- A great example is in the test_stepwise where at the top of the test file, you see a function that statically generates 2 different files to be tested. Example code of the this static data generation function is in the Test-case-analysis.md file.
### B. Dynamic Test Data
- I did not see any use of Dynamic Test Data in this project.
