# Branch naming conventions

## Name of the primary branch

Historically, the first branch of a newly created Git repository was called `master`. In recent years, many communities
and organizations have moved away from using that branch name because it is offensive to some people, switching
to `main`. As an example, consider [GitHub's advice on the topic](https://github.com/github/renaming).

We share those concerns, so the least we can do is to avoid the problematic term as much as possible.

Therefore, the following rules apply to the name of the primary or production branch:

1. In new repositories, no branch named `master` is created.
    - If there is one branch which contains the up-to-date production or release code (e.g. when using the Gitflow
      pattern), it should be called `main` instead.
    - Note that in some repositories, other naming/branching schemes may make more sense. There is no need to create an
      `main` branch that would not exist or be used otherwise.
2. For older repositories that have a `master` branch and are in active use, the branch should be renamed to `main` if
   possible.
    - If there are significant obstacles (such as CI systems we do not control) or the effort required for renaming
      cannot be justified, the team may decide to keep the old name. However, that decision must be documented in a
      README file in the repository and should be revisited regularly.
3. In discussions, we actively try to normalize always referring to the primary or production branch as `main`.
    - This means we neither say things like "master or main" nor do we keep saying "master" out of habit. Instead, we
      simply assume that any listener who does not find a branch with the literal name of `main` in their repository can
      map that name to `master`.
    - Furthermore, unless specifically discussing plans for (or the feasibility of) branch renaming in a repository,
      we do not mention or emphasize the name change. For example, we refrain from comments like "whatever your primary
      branch is called" (which are irrelevant at best and toxic at worst).

## Capitalization and format

Branch names use lowercase letters, digits and dashes. Slashes are used to separate branch type prefixes from the rest
of the name.
