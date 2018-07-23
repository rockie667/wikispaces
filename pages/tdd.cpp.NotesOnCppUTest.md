---
title: tdd.cpp.NotesOnCppUTest
---
# Using CppUTest in VisualStudio 2008
These do not include working in the express version. See the old section below for some additional details to get this working.

## First Time Only
* Download [[http://sourceforge.net/projects/cpputest/|CppUTest]].
* Extract the zip into some directory (e.g. C:\projects\CppTdd\CppUTest).
* Go into the CppUTest directory under which you extracted the zip file.
* Double-click on CppUTest.dsw.
** Do NOT click on CppUTest.dsp. That will leave out the self-tests.
* When asked to convert project, click yes.
* Execute the unit tests to make sure they work (ctrl-F5), you'll have some warnings in both the build and the output window. The warnings should all be related to the use of sprintf.

### Each Time You Want a New Solution
* Create a new solution with an Empty C++ Project (new, Empty C++ project)
* Add a source file (select project, right-click, add, new item..., c++ file) to your Empty C++ project called RunAllTests.cpp:
```cpp
# include <CppUTest/CommandLineTestRunner.h>

int main(int argc, char** argv) {
    return CommandLineTestRunner::RunAllTests(argc, argv);
}
```
* Select the project (not the solution), right-click, select properites
* On the  C/C++ tab, add the include directory under where you extracted the CppUTest.zip file (e.g., C:\projects\CppTdd\CppUTest\include)
* On the Linker:Input tab, enter two new additional dependencies: winmm.lib CppUTest.lib
* On the Linker:General tab, add the lib directory under where you extracted the CppUTest.zip file (e.g., C:\projects\CppTdd\CppUTest\lib)
* Verify your project builds (ctrl-b)
* Add a test file. Create a new source file. E.g. FooTest.cpp:
```cpp
// the below include should be the last include if you are using <vector>, <string>, etc.
# include <CppUTest/TestHarness.h>

TEST_GROUP(FooTest) {
};

TEST(FooTest, TestName) {
  LONGS_EQUAL(1, 1);
}
```
* Make sure your test runs, hit ctrl-F5. You should see something like:
```
.
OK (1 tests, 1 ran, 1 checks, 0 ignored, 0 filtered out, 0 ms)

Press any key to continue . . .
```

## Working With Boost
* Build or Install the boost library
* Include the distrubtion directory, e.g. C:\boost\boost_1_38
* When referring to libs, #include <boost/...>
* Add the libs directory in the Properties:Linker:General:Additional Library Directories, e.g. C:\boost\boost_1_38\lib

----
----
old
----
----
### First Time Only
#### Install CppUTest
* Download [[http://sourceforge.net/projects/cpputest/|CppUTest]].
* Extract the zip into some directory.
* Go into the CppUTest directory under which you extracted the zip file.
* Double-click on CppUTest.dsw.
** Do NOT click on CppUTest.dsp. That will leave out the self-tests.
* When asked to convert project, click yes.
* Execute the unit tests to make sure they work (ctrl-F5), you'll have some warnings in both the build and the output window. The warnings should all be related to the use of sprintf.
** If you use VS2005 Express, it won't have windows.h, which is needed for CppUtest.
*** Google for PSDK-x86.exe and VS2005 and see all the complaints and how to fix.

#### Install DevExpress Refactor!
* Install [[http://devexpress.com/Products/Visual_Studio_Add-in/RefactorCPP/|DevExpress Refactor!]] - This is a free refactoring tool for C++ in Developer Studio.
* Note: You'll need to exit Developer Studio to install this product.
* This tool has caused problems at times. When it works, it is a big help. If you experience problems with your IDE freezing, uninstall this to see if the problems go away.

### Each Time You Want a New Solution
* Create a new solution with an Empty C++ Project
* Select the solution and select Add:Existing Project
* Select the solution created when converting CppUTest
* (Optional) Remove the AllTests project included when you included the solution
* Add a source file to your Empty C++ project called AllTests.cpp:
```cpp
# include <CppUTest/CommandLineTestRunner.h>

int main(int ac, char** av)
{
    return CommandLineTestRunner::RunAllTests(ac, av);
}
```
* Make your project dependent on CppUTest(Solution, Project Dependencies... )
* Add the CppUTest include directory to your C++ project
> <path of CppUTest project>\include
** (Use your project's Properties, C/C++, General, Additional Include Directories...)
* Add one library to be linked in (CppUTest uses a time function found here):
> winmm.lib
** (project Properties... Linker, Input, Additional Dependencies)
* You should be able to start writing and executing tests. E.g., Add a file to hold some tests, FooTest.cpp
```cpp
# include <CppUTest/TestHarness.h>

TEST_GROUP(FooTest)
{
};

TEST(FooTest, TestName) {
  LONGS_EQUAL(1, 1);
}
```
## Working With Boost
* Build or Install the boost library
* Include the distrubtion directory, e.g. C:\boost\boost_1_38
* When referring to libs, #include <boost/...>
* Add the libs directory in the Properties:Linker:General:Additional Library Directories, e.g. C:\boost\boost_1_38\lib