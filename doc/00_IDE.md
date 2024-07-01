# IDE

## What is an IDE?

An integrated development environment (IDE) is a software application that helps programmers develop software code efficiently. It increases developer productivity by combining capabilities such as software editing, building, testing, and packaging in an easy-to-use application.
The IDE is one of the most important tool for the developer. IDE are customizable and it takes some time before finding what works best.
The purpose of this guide is to provide a basic setup of Visual Studio Code to get started for a new project.

## Visual Studio Code

Link to download and install VS Code <https://code.visualstudio.com/>.
VSC provides:
 1. syntax highlight, autocomplete, linting
 2. debug facilities
 3. smooth git integrarion (source control)
 4. customization through plugins and extensions

## Linting

Linting is the automated checking of your source code for programmatic and stylistic errors. This is done by using a lint tool (otherwise known as linter). A lint tool is a basic static code analyzer. The term linting originally comes from a Unix utility for C.
Linting tells you haed-of-time if your code has potential errors and if your coding style is not correct. It is common practice to _enforce_ linter checks in your workflow (e.g. by doing lint checks on the code you commit to git)
Suggested linters are:
 - C/C++ : Clang format
 - Python : Fluke8, pylint, Mypy (static type checks), bandit (security)
 - Verilog/ SystemVerilog : Verible
 - YAML : yamllint

There are also other linters that check for trailing spaces and wrong file format (e.g. editorconfig-checker) 

## AI assisted autocomplete

We live in AI era and learning how to use AI is fundamental.
AI can make your code writing faster and suggest new approaches/solutions.
A good and free for student AI assistant is the GitHub Copilot <https://github.com/features/copilot>, available as a plugin.

## SSH remote control

In most workplace, the engineer receives a small laptop with limited resources. The laptop works as an access point to a more powerfull server where the actual work is done.
The "Remote - SSH" plugin allows to work in the local machine as if you where on the actual sever through SSH. Please learn how to setup an SSH-key to make this more efficient.
