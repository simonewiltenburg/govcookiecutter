# Pre-commit hooks

This repository uses the Python package [`pre-commit`][pre-commit] to manage pre-commit hooks. Pre-commit hooks are
actions which are run automatically, typically on each commit, to perform some common set of tasks. For example, a
pre-commit hook might be used to run any code linting automatically, providing any warnings before code is committed,
ensuring that all of our code adheres to a certain quality standard.

```{contents}
:local:
:depth: 2
```

## Purpose

For this repository, we are using `pre-commit` for a number of purposes:

- Checking for secrets being committed accidentally — see [here](#definition-of-a-secret-according-to-detect-secrets)
  for the definition of a "secret";
- Checking for any large files (over 5 MB) being committed; and
- Cleaning Jupyter notebooks, which means removing all outputs and execution counts.

We have configured `pre-commit` to run automatically on _every commit_. By running on each commit, we ensure that
`pre-commit` will be able to detect all contraventions and keep our repository in a healthy state.

## Installation

In order for `pre-commit` to run, action is needed to configure it on your system.

- Install the `pre-commit` package into your Python environment from `requirements.txt`; and
- Run `pre-commit install` in your terminal to set up `pre-commit` to run when code is _committed_.

## Using the `detect-secrets` pre-commit hook

> ⚠️ The `detect-secrets` package does its best to prevent accidental committing of secrets, but it can't catch
> everything. **It doesn't replace good software development practices!** See the definition of a secret
> [here](#definition-of-a-secret-according-to-detect-secrets) for further information.

We use [`detect-secrets`][detect-secrets] to check that no secrets, as defined
[here](#definition-of-a-secret-according-to-detect-secrets), are accidentally committed. This hook requires you to
generate a baseline file if one is not already present within the root directory. To create the baseline file, run the
following at the root of the repository:

```shell
detect-secrets scan > .secrets.baseline
```

Next, audit the baseline that has been generated by running:

```shell
detect-secrets audit .secrets.baseline
```

When you run this command, you'll enter an interactive console and be presented with a list of high-entropy string
and/or anything which _could_ be a secret, and asked to verify whether this is the case. By doing this, the hook will
be in a position to know if you're later committing any _new_ secrets to the repository, and it will be able to alert
you accordingly.

### Definition of a "secret" according to `detect-secrets`

The [`detect-secrets` documentation][detect-secrets], as of January 2021, says it works:

> ...by running periodic diff outputs against heuristically crafted \[regular expression\] statements, to identify
> whether any new secret has been committed.

This means it uses regular expression patterns to scan your code changes for anything that **looks like a secret
according to one or more of these regular expression patterns**. By definition, there are only a limited number of
patterns, so the `detect-secrets` package cannot detect every conceivable type of secret.

To understand what types of secrets will be detected, read the [caveats][detect-secrets-caveats], and the list of
[supported plugins][detect-secrets-plugins] that the package uses. Also, you should use secret variable names that
contain words that will trip the KeywordDetector plugin; see the `DENYLIST` variable
[here][detect-secrets-keyword-detector] for the full list of words.

### If `pre-commit` detects secrets during commit

If `pre-commit` detects any secrets when you try to create a commit, it will detail what it found and where to go to
check the secret.

If the detected secret is a false-positive, you should update the `.secrets.baseline` through the following steps:

- Run `detect-secrets scan --update .secrets.baseline` from the root folder in the terminal to index the
  false-positive(s);
- Next, audit all indexed secrets via `detect-secrets audit .secrets.baseline` (the same as during initial set-up, if a
  `.secrets.baseline` doesn't exist); and
- Finally, ensure that you commit the updated `.secrets.baseline` in the same commit as the false-positive(s) it has
  been updated for.

If the detected secret is actually a secret (or other sensitive information), remove the secret and re-commit. There is
no need to update the `.secrets.baseline` file in this case.

If your commit contains a mixture of false-positives and actual secrets, remove the actual secrets first before
updating and auditing the `.secrets.baseline` file.

## Keeping specific Jupyter notebook outputs

It may be necessary or useful to keep certain output cells of a Jupyter notebook, for example charts or graphs
visualising some set of data. To do this, add the following comment at the top of the input block:

```julia
# [keep_output]
```

This will tell the hook not to strip the resulting output of this cell, allowing it to be committed.

[detect-secrets]: https://github.com/Yelp/detect-secrets
[detect-secrets-caveats]: https://github.com/Yelp/detect-secrets#caveats
[detect-secrets-keyword-detector]: https://github.com/Yelp/detect-secrets/blob/master/detect_secrets/plugins/keyword.py
[detect-secrets-plugins]: https://github.com/Yelp/detect-secrets#currently-supported-plugins
[pre-commit]: https://pre-commit.com/