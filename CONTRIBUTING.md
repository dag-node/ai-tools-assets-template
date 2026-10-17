# Contributing

A set lives in `sets/<set>/` and follows the [asset
format](https://github.com/dag-node/ai-tools-assets-tools/blob/main/format/FORMAT.md)
of ai-tools-assets-tools. Check a change before pushing, with the tools
checked out beside this repository:

```bash
python3 ../ai-tools-assets-tools/tools/validate
```

## Branches and pull requests

Branch off `develop` and open the pull request into `develop`; it is
rebase-merged once `validate` is green. `main` receives releases, through
a `Merge develop` pull request, as the [release
page](https://github.com/dag-node/ai-tools-assets/blob/main/docs/development/release.md)
of dag-node/ai-tools-assets states.

## Sign-off

Every commit carries a `Signed-off-by: Name <email>` trailer that matches its
author; `git commit -s` adds it. The trailer certifies the [Developer
Certificate of Origin 1.1](https://developercertificate.org/): that you wrote
the change, or have the right to submit it, under the licence the file states.
The `validate` workflow refuses a pull request holding a commit without one.
