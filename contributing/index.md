---
layout: page
title: Contributing
---

# Contributing

### Contributor Conduct

This is a project for the community, by the community!

In order to foster a welcoming and inclusive open source community, please follow the guidelines in the 
[Contributor Code of Conduct]({% link contributing/code_of_conduct.md %}). By participating in the Observatory Control System community, 
you agree to abide by its terms.

### Testing

The codebases of the OCS projects all have test suites, which are vital for checking that
the project works as intended. The projects all have varying degrees of test coverage. Each new
feature or fix should have a test confirming its functionality. An automated check will make sure
that test coverage does not decrease over time.

Besides running test suites, it is extremely useful for development to be able to run a project locally
on a development machine. Instructions are provided in each project's README on how to run that project locally.

### Code quality, guidelines, and style

The projects that are all written in Python should use at least version 3.7. Stylistically,
all projects should conform to the [PEP8](https://www.python.org/dev/peps/pep-0008/) standard.
New code must pass a static code analysis check, which is automatically run when new code is pushed
to GitHub.
