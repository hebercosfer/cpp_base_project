# cpp_base_project
A base CPP project, with a CMake structure and tools for building/maintaining a project.

## Clang-Format and Clang-tidy
On this project we are using Clang tools that format and do static code analysis in your C/C++ files.
More information about them can be found on: https://llvm.org/

In the root of this project you can find a .clang-format file that define the Coding Guidelines used in the current project.
This configuration can be changed and you can find more information here: https://clang.llvm.org/docs/ClangFormatStyleOptions.html
The benefit of using clang-format is quite clear, as you automatize the coding format check of your project, reducing the time spent on Code Reviews.
Clang-format automatically formats your code using IDE plugins or through pre-commit hook.

Also in the root of this project you can find a .clang-tidy file that configures how would your project be analysed statically with clang-tidy.
The configuration can also be changed according to your project requirements, and you can find the checks available here: https://clang.llvm.org/extra/clang-tidy/checks/list.html
Clang-tidy works as a static code analysis and can diagnose and fix typical programming errors, identifying potential bugs and bad smell code.
Clang-tidy can be executed automatically with your configuration file through an IDE plugin or a pre-commit hook.