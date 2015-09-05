Welcome to the Stan wiki. This page is primarly inteded for developers, both active and new to Stan.

### Contents
* [General Notes for Developers](#general-notes-for-developer-process)
* [Model Concept](#model-concept)
* [Feature Design Docs](#stan-feature-specs-and-design-docs)
* [Long Term To-Do List](#long-term-to-do-list)

---

### General Notes for Developer Process

[//]: # (DL: Rearrange. I think it makes sense to split this section into)
[//]: # (DL:   two parts: common information to be accessed by current)
[//]: # (DL:   developers, a second part for process that's useful for )
[//]: # (DL:   new developers)

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
* [OS and Platform Detection Macros for C++](wiki/Compiler-and-OS-Detection-Macros-for-Cpp)

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

* [Ragged array spec](wiki/Ragged-array-spec)


---

### Long Term To-Do List

This is complementary to the issue tracker of smaller tidbits:

* [Longer-Term To-Do List](wiki/Longer-Term-To-Do-List)

### C++11 Migration Plan

* [C++11 Migration Plan](wiki/Cpp11-Upgrade)

### Out-of-Date To-Do List

GitHub's issue tracker is more up to date and contains a `project` tag if you're looking for projects. An older list of to-do items, with some more ambitious projects, is here:

* [Stan To-Do List](wiki/To-Do-List)
