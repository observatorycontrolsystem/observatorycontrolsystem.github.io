---
layout: page
title: Contributor Guidelines
---

# Contributing
This page will go over the process for contributing to the Observatory Control System. We appreciate your interest in contributing to this project - this has always been a project for the community, by the community!

## Contributing Code/Documentation

If you’re interested in contributing code to the project, thank you! For those unfamiliar with the process of contributing to an open-source project, you may want to read through Github’s own short informational section on [how to submit a contribution.](https://opensource.guide/how-to-contribute/#how-to-submit-a-contribution)

## Familiarizing yourself with Git

If you are not familiar with git, we encourage you to briefly look at the [Git Basics](https://git-scm.com/book/en/v2/Git-Basics-Getting-a-Git-Repository) page.

## Git Workflow

The workflow for submitting a code change is, more or less, the following:

1. Fork the appropriate repository to your own Github account.
![Fork Repository](/assets/images/fork_repo.png)

2. Clone the forked repository to your local working machine: 
```
git clone git@github.com:<Your Username>/<Repository Name>.git
```

3. Add the original “upstream” repository as a remote.
```
git remote add upstream https://github.com/observatorycontrolsystem/<Repository Name>.git
```

4. Ensure that you’re synchronizing your repository with the “upstream” one relatively frequently.
```
git fetch upstream
git merge upstream/main
```

5. Create and checkout a branch for your changes (see [Branch Naming](#branch-naming)).
```
git checkout -b <New Branch Name>
```

6. Commit frequently, and push your changes to Github. Be sure to merge main in before submitting your pull request.
```
git push origin <Branch Name>
```

7. When your code is complete and tested, create a pull request from the upstream Observatory Control System repository:
![New Pull Request](/assets/images/new_pull_request.png)


Be sure to click “compare across forks” in order to see your branch! 

![Compare Across Forks](/assets/images/compare_across_forks.png)

We may ask for some updates to your pull request, so revise as necessary and push when revisions are complete. This will automatically update your pull request.

## Branch Naming

Branch names should be prefixed with the purpose of the branch, be it a bugfix or an enhancement, along with a descriptive title for the branch.

* `bugfix/fix-typo-target-detail`
* `feature/reticulating-splines`
* `enhancement/refactor-planning-tool`

## Code Style
We recommend that you use a linter, as all pull requests must pass a Codacy check. We also recommend configuring your editor to automatically remove trailing whitespace, add newlines on save, and other such helpful style corrections.

## What does the Code of Conduct mean for me?
Our [Code of Conduct]({% link contributing/code_of_conduct.md %}) means that you are responsible for treating everyone on the project with respect and courtesy regardless of their identity. If you are the victim of any inappropriate behavior or comments as described in our Code of Conduct, we are here for you and will do the best to ensure that the abuser is reprimanded appropriately, per our code.
