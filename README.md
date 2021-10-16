# Poor Mans Line Coverage In C
An hack using the preprocessor to get (non-branching) line code coverage

# Idea

Why not use the __LINE__ preprocessor directive to keep track of what lines have been executed?

Let's say we have simple function like this:

  ```
  void maybe_trigger(float temperature) {
      if(temperature > 100) {
          printf("Triggered at value %d.\n", a);
      }
      else {
          printf(Not triggered yet.\n");
      }
  }
  ```

We want to write unit tests to cover as much as possible of this function, be we don't have the
tooling available. What can we do?

Well, we can define a macro C (for cover) like this:

  ```
  #define C  (rowsVisited[numRowsVisited++] = __LINE__);
  ```

`rowsVisited` together with `numRowsVisited` keeps track of what lines are visited, using the
__LINE__ preprocessor macro. They can e.g be defined just above the function we want to cover.

Then we can put this C macro at the start of every line we want to cover:

  ```
  void maybe_trigger(float temperature) {
  C    if(temperature > 100) {
  C        printf("Triggered at value %d.\n", a);
  C    }
  C    else {
  C        printf(Not triggered yet.\n");
  C    }
  }
  ```

Now we can write unit tests, which will update the rowsVisited buffer, and using this
little helper function, we can display what coverage we have in percent, and what lines
are missing:

   ```
   void coverageReport(int scopeStart, int scopeEnd) {
      int totalLines = scopeEnd - scopeStart + 1;
      int numCovered = 0;
      for(int line = scopeStart; line <= scopeEnd; line++) {
        int covered = 0;
        for(int i=0; i<numRowsVisited; i++) {
          if(rowsVisited[i] == line) {
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
   
Note that `coverageReport` needs to know line to start- and end with. For example, maybe
the above function was defined at line 37-42 in some module, then it would be called like this:

   ```
   coverageReport(37, 42);
   ```

One final challange: when to call coverageReport? Well, one idea is from a 'reportTest' unit test,
which is somehow run last, after all other (relevant) unit tests. In CGreen[1] style:

   ```
   Ensure(some_suite_name, reportTest) {
     coverageReport(37, 42);
   }
   ```


1. https://cgreen-devs.github.io/
