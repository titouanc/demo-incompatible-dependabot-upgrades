This repo demonstrates how dependabot generates Python package upgrades that
are incompatible with the requirements declared in setup.cfg

# Test setup

This repo emulates a simple Python package that only depends on tensorflow, a
popular Python lib for machine learning _(but really the dependency used as
example does not matter for the demonstration)_

Our package specify its dependency in [`setup.cfg`](setup.cfg), which is one
of the many ways to declare a Python package and its requirements. We do not
impose any version constraint on the dependencies here.

### Freezing (or "locking", "pinning") dependencies

We then generate a "lockfile" of our package's requirements with
[pip-tools](https://github.com/jazzband/pip-tools):

```bash
pip-compile setup.cfg
```

This gives us a file [`requirements.txt`](requirements.txt) where our package
dependencies, as well as all their transitive dependencies are pinned to a
specific version (`pkg==version`).

### Continuous integration

Github Actions is set up to install the dependencies from requirements.txt,
and then run `pip-check` to ensure that all requirements are correctly
satisfied.

### Dependabot

Dependabot is enabled for the pip ecosystem on this repository,
see [dependabot.yml](.github/dependabot.yml).


# Observed results

In this setup, we see that Dependabot open some PRs. For example:
https://github.com/titouanc/demo-incompatible-dependabot-upgrades/pull/1

In this PR, Dependabot is upgrading numpy to v1.21, but tensorflow requires
numpy v1.19 (and not higher).

This leads to a broken state, because there are some dependency conflicts, as
the CI catches while installing the dependencies:
https://github.com/titouanc/demo-incompatible-dependabot-upgrades/runs/3678157870#step:4:124

## Practical considerations

As we can see in the build logs, pip's backtracking algorithm is trying to find
a solution to the dependency conflict, ultimately determines that no solution
exists, and eventually raises an error.

However, real world Python libraries and applications have much larger
requirements lists, and the pip backtracking algorithm may take much more
longer, **up to several hours (!)**, to raise this error. Therefore, the CI
environment could even timeout before the actual error occurs.


# Does not happen with requirements.in

If we specify the requirements in a file `requirements.in` instead of
`setup.cfg`, and then compile it like this:

```bash
pip-compile requirements.in
```

Then, Dependabot does not open any problematic pull request.
