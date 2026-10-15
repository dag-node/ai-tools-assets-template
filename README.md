# ai-tools-assets-template

**A starting point for publishing your own sets of skills and subagents for
coding agents on ai-tools hosts.**

**Status.** In preparation: the [layout](#layout) passes the pull-request checks,
and the publishing guide is not written yet. Do not generate a repository
from it yet. [ai-tools-assets](https://github.com/dag-node/ai-tools-assets) is
a working publisher repository in the meantime.

## Layout

```text
publisher.conf                  who publishes: replace each change-me
sets/change-me-example/         one example set holding one example skill
.claude-plugin/marketplace.json written by sync-manifests
.github/workflows/              validate and release, calling ai-tools-assets-tools
LICENSE, LICENSES/, REUSE.toml  MIT-0 for this skeleton
```

## Getting started

Clone [ai-tools-assets-tools](https://github.com/dag-node/ai-tools-assets-tools)
beside your repository, then check your tree:

```bash
python3 ../ai-tools-assets-tools/tools/validate
```

## Licence

The skeleton is [MIT-0](LICENSE): copy it, and replace the notice with your
own, without attribution.
