The Stan Wiki here is primarily intended for developers, both active and new to Stan.

### Contents
* [General Notes for Developers](#general-notes-for-developer-process)
* [Long Term To-Do List](#long-term-to-do-list)
* [Automatic Differentiation](#automatic-differentiation)
* [Model Concept](#model-concept)
* [Feature Design Docs](#stan-feature-specs-and-design-docs)

---

### General Notes for Developer Process

The Git and code review process used for developers and some useful tools:

* [Developer Process](wiki/Developer-Process)
* [Pull Request Template](wiki/Pull-Request-Template)
* [Unit Testing with Python](wiki/Testing-Stan-using-Gnu-Make-and-Python)
* [Continuous Integration Testing](wiki/Continuous-Integration)
* [How to Write Unit Tests](wiki/How-to-Write-Unit-Tests-with-GoogleTest)
* [Code Quality Requirements](wiki/Code-Quality)
* [Coding Style and Idioms](wiki/Coding-Style-and-Idioms)
* [How to Contribute a New Function to Stan](wiki/Contributing-New-Functions-to-Stan)
* [Developer Tools and Tricks](wiki/Developer-Tricks)
* [CmdStan developers under Windows: additional dependency](wiki/CmdStan-developers-under-Windows:-additional-dependency)
* [OS and Platform Detection Macros for C++](wiki/Compiler-and-OS-Detection-Macros-for-Cpp)

### Long Term To-Do List

This is complementary to the issue tracker of smaller tidbits:

* [Longer-Term To-Do List](wiki/Longer-Term-To-Do-List)

### C++11 Migration Plan

* [C++11 Migration Plan](wiki/Cpp11-Upgrade)

---

### Automatic Differentiation

A preliminary stab at some doc on using automatic differentiation, aimed at both Stan developers and those who want to use just Stan's automatic differentiation in some other context. 

* [Automatic Differentiation](wiki/Automatic-Differentiation-API)

---

### Model Concept

Explanation of the code generated for a model:

* [Stan Model Concept](wiki/Model-Concept) 

---

### Stan Feature Specs and Design Docs

Specs and design docs and for Stan features, as originally proposed.  See Stan developer group mailing list for more discussion around specific features.

* [Functions Design](wiki/Function-Syntax-and-Semantics-Design)

* [ODE Design](wiki/ODE-Integrator-Support)

* [MLE and MML Design](wiki/MLE-and-MML-Design)

* [Command Refactor Spec](wiki/Stan-API-Refactor)

* [Testing framework](wiki/Testing-framework)

### Out-of-Date To-Do List

GitHub's issue tracker is more up to date and contains a `project` tag if you're looking for projects. An older list of to-do items, with some more ambitious projects, is here:

* [Stan To-Do List](wiki/To-Do-List)