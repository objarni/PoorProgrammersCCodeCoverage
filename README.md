# Poor Programmers C Code Coverage
A hack using the preprocessor to get (non-branching) code coverage.

# Idea

Why not use the `__LINE__` preprocessor directive to keep track of what lines have been executed?

Let's say we have simple function like this:

  ```cpp
  void maybe_trigger(float temperature) {
      if(temperature > 100) {
          printf("Triggered at value %d.\n", temperature);
      }
      else {
          printf(Not triggered yet.\n");
      }
  }
  ```

We want to write unit tests to cover as much as possible of this function, be we don't have the
tooling available. What can we do?

Well, we can define a macro C (for cover) like this:

  ```cpp
  #define C  (lineVisited[numLinesVisited++] = __LINE__);
  int lineVisited[50000];
  int numLinesVisited = 0;
  ```

`lineVisited` together with `numLinesVisited` keeps track of what lines are visited, using the
`__LINE__` preprocessor macro. They can e.g be defined just above the function we want to cover.

Then we can put this C macro at the start of every line we want to cover:

  ```cpp
  void maybe_trigger(float temperature) {
  C    if(temperature > 100) {
  C        printf("Triggered at value %d.\n", a);
  C    }
  C    else {
  C        printf(Not triggered yet.\n");
  C    }
  }
  ```

Now we can write unit tests, which will update the lineVisited buffer, and using this
little helper function, we can display what coverage we have in percent, and what lines
are missing:

   ```cpp
   void coverageReport(int lineStart, int lineEnd) {
      int totalLines = lineEnd - lineStart + 1;
      int numCovered = 0;
      for(int line = lineStart; line <= lineEnd; line++) {
        int covered = 0;
        for(int i=0; i<numLinesVisited; i++) {
          if(lineVisited[i] == line) {
            covered = 1;
            numCovered++;
            break;
          }
        }
        if(!covered)
          printf("Line %d was not covered.\n", line);
      }
      printf("Coverage: %1.1f\n", 100 * numCovered / totalLines);
   }
   ```
   
Note that `coverageReport` needs to know what line to start and end with. For example, maybe
the above function was defined at line 37-42 in some module, then it would be called like this:

   ```cpp
   coverageReport(37, 42);
   ```

One final challange: when to call coverageReport? Well, one idea is from a 'reportTest' unit test,
which is somehow run last, after all other (relevant) unit tests. In CGreen[1] style:

   ```cpp
   Ensure(some_suite_name, reportTest) {
     coverageReport(37, 42);
   }
   ```


1. https://cgreen-devs.github.io/
