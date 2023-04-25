# CPP Base Project

A base CPP project, with a CMake structure and tools for building/maintaining a project.

## Clang-Format and Clang-tidy

On this project we are using Clang tools that format and do static code analysis in your C/C++
files.
More information about them can be found on: <https://llvm.org/>

In the root of this project you can find a .clang-format file that define the Coding Guidelines
used in the current project.
This configuration can be changed and you can find more information here:
<https://clang.llvm.org/docs/ClangFormatStyleOptions.html>
The benefit of using clang-format is quite clear, as you automate the coding format check of your
project, reducing the time spent on Code Reviews.
Clang-format automatically formats your code using IDE plugins or through pre-commit hook.

Also in the root of this project you can find a .clang-tidy file that configures how would your
project be analysed statically with clang-tidy.
The configuration can also be changed according to your project requirements, and you can find the
checks available here: <https://clang.llvm.org/extra/clang-tidy/checks/list.html>
Clang-tidy works as a static code analysis and can diagnose and fix typical programming errors,
identifying potential bugs and bad smell code. Clang-tidy can be executed automatically with your
configuration file through an IDE plugin or a pre-commit hook.

## Pre-Commit hooks

Git hook scripts are useful for identifying simple issues before submission to code review and the
pre-commit hooks check the modified files before the commit is executed. More information on:
<https://pre-commit.com>

In the root of this project you can find the .pre-commit-config.yaml that configures that add and
configures the hooks. Here are the supported hooks you can add on the configuration file:
<https://pre-commit.com/hooks.html>

The goal of the pre-commit hooks is to do not allow commits that breaks the consistency of the
project, fixing the modified files or suggesting fixes. Projects with many contributors can benefit
highly from it, so It can keep the same code format and quality, for C++ files, as well as for
Python, CMake files, Yaml files, Markdown documents, Json files, and others.

This project uses the repositories to access the hooks, but a different configuration to use local
installation can be done, and is usable on a protected network that can't access the internet. When
using local, you may need to change the language of the hook to system.

### Hooks Description

trailing-whitespace: Trims trailing whitespace that can be introduced by mistake or by different
editors.

end-of-file-fixer: ensures that a file is either empty, or ends with one newline.

check-added-large-files: prevents giant files from being committed.

check-case-conflict: checks for files that would conflict in case-insensitive filesystems.

check-yaml: checks yaml files for parseable syntax. Yaml files are configuration files, one of them
is the pre-commit configuration file, and this hook keep the Yaml files with a consistent format.

check-json: checks json files for parseable syntax.

clang-format and clang-tidy: Mentioned in the section above.

pylint: Pylint is a static code analyser for Python files. The file .pylintrc configures the flags
for the analyser to check and to format the Python scripts.

autopep8: autopep8 automatically formats Python code to conform to the PEP 8 style guide,
maintaining a consistent code format for python files in the project. The configuration file for
this hook is in the .pep8 found on the root of the project. How to modify this configuration could
be found here: <https://github.com/hhatto/autopep8#id8>

codespell: Checks for common misspellings in text files. It does not modify the files but suggest
correct words to be used. The file .codespellrc located on the root of the project configures this
hook and words to be ignored can be added there.

cmake-lint: will check your listfiles for style violations, common mistakes, and anti-patterns.

cmake-format: can format your listfiles nicely so that they can keep the same code style. The python
script .cmake-format.py located on the root configures the cmake-format and cmake-lint hooks. Here
you can find how to modify this configuration:
<https://cmake-format.readthedocs.io/en/latest/configuration.html>

markdownlint: Check markdown files and flag style issues. Markdownlint hook uses the file .mdlrc
and .mdlrc.rb to configure how to check the MD files. Here you can find how to modify the MD
styles: <https://github.com/markdownlint/markdownlint/blob/main/docs/creating_styles.md>

### Installation of Pre-commit Hooks

For installation of the pre-commit process and some of its hooks, you will need to install Python
and using the Package Manager for Python (pip) to install some of the hooks. The file
requirements-dev.txt contains the packages required for this project, including the pre_commit and
pre-commit-hooks, you can install them with the command `pip install -r requirements-dev.txt`. If
your project would need other packages that can be installed with pip, they can be added on the
file for an easy installation process.

For the markdownlint hook, it's needed to have a Ruby installation that you can find here:
<https://www.ruby-lang.org/en/downloads/>
After installing it, you would be able to install the markdownlint using the Ruby Package Manager,
called gem. So, execute: `gem install mdl`

Having the Python Scripts folder in your PATH is needed, as well as the Ruby bin folder are needed.

The last step to have your pre-commit automatically checking your commit, is to install the
pre-commit on your local repository with the command: `pre-commit install`
After that, when you commit your changes, you should see the pre-commit checks been done.
