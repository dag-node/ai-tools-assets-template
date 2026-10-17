# ai-tools-assets-template

**A starting point for publishing your own sets of skills and subagents for
coding agents on ai-tools hosts.**

A repository generated from this template, and initialized with
`new-publisher`, passes the pull-request checks and the release check of
[ai-tools-assets-tools](https://github.com/dag-node/ai-tools-assets-tools)
under your own name. [ai-tools-assets](https://github.com/dag-node/ai-tools-assets)
is a publisher repository made the same way.

## Quick start

```bash
git clone https://github.com/<owner>/<repo>
git clone https://github.com/dag-node/ai-tools-assets-tools
cd <repo>
python3 ../ai-tools-assets-tools/tools/new-publisher --repository <owner>/<repo> --contact <email>
```

1. Press **Use this template** and create `<owner>/<repo>` with the defaults,
   which copy `main` alone.
2. Clone it, and the tools beside it, with the commands in the block. A host with the
   `ai-tools-assets-tools` RPM installed runs the commands from
   `/usr/share/ai-tools-assets-tools/tools/` instead.
3. Run `new-publisher`, then the git lines it prints, which commit the
   initialization on `main` and create `develop` from it. The command
   writes `publisher.conf`, `LICENSE`, `REUSE.toml`, `CODEOWNERS`, this
   README and the workflows' job names, and replaces the example set with
   your first set. Add `--tag-signer <fingerprint>` and
   `--artifact-signer <fingerprint>` to set the release signers in the same
   run, as [Release signing](#release-signing) states;
   `python3 ../ai-tools-assets-tools/tools/new-publisher --help` lists the
   other options.
4. Apply the settings a copy does not inherit from the template:
   - [ ] **Private vulnerability reporting**, the route `SECURITY.md` names.
   - [ ] A **`release` environment** whose deployment policy allows tags
         `*/v*` and no branch, holding the secrets `GPG_SIGNING_KEY` and
         `GPG_SIGNING_PASSPHRASE`, as [Release signing](#release-signing)
         states.
   - [ ] The **rulesets** the [release
         page](https://github.com/dag-node/ai-tools-assets/blob/main/docs/development/release.md)
         of ai-tools-assets lists, with the required checks
         `<repo> / validate`, `<repo> / tests (python 3.9)` and
         `<repo> / tests (python 3.12)`.
   - [ ] **Actions**: require actions to be pinned to a full-length commit
         SHA, and set the workflow permissions to read repository contents
         and packages.
   - [ ] **Require contributors to sign off on web-based commits**, the
         DCO sign-off `CONTRIBUTING.md` asks for.
5. Release a set as that release page states. `repin-tools` moves the tools
   pin from your `develop`.

## Release signing

A release through GitHub is signed with two keys, and neither defaults to
the other:

| | `--tag-signer` | `--artifact-signer` |
|---|---|---|
| `release.yml` input | `tag-signer-fingerprints` | `artifact-signer-fingerprint` |
| Signs | the release tag `<set>/v<semver>`, on a maintainer's workstation | the zip, `SHA256SUMS` and the RPMs, in the release job |
| Secret key | in the maintainer's own keyring; the workflow holds the fingerprint alone | in the secret `GPG_SIGNING_KEY` of the `release` environment |
| The release job checks | that the tag is signed by a primary on the list, whose public keys it fetches from `tag-signer-key-url` | that the key in `GPG_SIGNING_KEY` has this primary |
| How many | one per maintainer who may tag; repeat the option | one |

An option left out keeps the template's placeholder, and a tag pushed then
stops at the tag check. `--tag-signer` sets `tag-signer-key-url` to
`https://github.com/<owner>.gpg` unless `--tag-signer-key-url` names another
URL. Keep the two keys apart: the artifact key's secret sits in CI, and the
tag key's does not, so a leaked CI secret does not publish a release by
itself.

**The `release` environment is required for a release through GitHub.**
`release.yml` calls the tools' `set-release.yml`, whose job runs in the
environment named `release` and reads both secrets from it. Create it before
the first tag: where it is missing, GitHub creates it at the first run with
no protection rule and no secret, and the release stops at the step that
checks the signing key, before anything is signed. A publisher who releases
[without GitHub](#without-github) does not push a `<set>/v<semver>` tag and
does not need the environment.

**`GPG_SIGNING_KEY` is ideally a signing subkey** of the artifact signer,
exported without the primary's secret, which stays offline:

```bash
gpg --armor --export-secret-subkeys '<subkey fingerprint>!' | gh secret set GPG_SIGNING_KEY --env release
gh secret set GPG_SIGNING_PASSPHRASE --env release
```

`GPG_SIGNING_PASSPHRASE` is the passphrase that protects the exported
subkey. The release job and each host's binding pin the primary fingerprint,
so a new signing subkey under the same primary rotates the old one without
a change to `release.yml` or to a binding; a host verifies the new subkey's
signatures once its keyring holds the public key re-exported with it.
A new primary is a new key: the release job refuses it until
`artifact-signer-fingerprint` names it, and a host until its administrator
rebinds the set.

## Without GitHub

A publisher whose hosts take its sets from its own workstation, offline or
air-gapped, builds and signs a set locally and copies it to each host:

```bash
python3 ../ai-tools-assets-tools/tools/new-publisher --repository <owner>/<repo> --contact <email> --source <url>
python3 ../ai-tools-assets-tools/tools/new-asset <set> skill <set>-<name> --description "<what it does, then when to use it>"
python3 ../ai-tools-assets-tools/tools/build-set <set>
python3 ../ai-tools-assets-tools/tools/sign-set <set> --key <fingerprint>
gpg --armor --export <fingerprint> > keys/<fingerprint>.asc
python3 ../ai-tools-assets-tools/tools/verify-set build/<set> --key keys/<fingerprint>.asc --signer <fingerprint> --source .
```

`--repository` takes `<publisher>/<name>` in GitHub's form, and no repository
needs to exist: the command does not contact GitHub. The owner, lower-cased,
is the publisher and the first set's name, which a host's binding names; the
name is the package name in `REUSE.toml`. `--source` names where the sets
come from, in place of the GitHub URL. The README the command writes names
a GitHub marketplace; replace those lines with your own route.
`new-set <set>` adds a further set. `sign-set` signs the staged
`build/<set>/SHA256SUMS` with your own `gpg` key and prints the primary
fingerprint a host's binding names. `verify-set` runs the check a host makes
before it links a set, and with `--source .` rebuilds the set from this
tree and refuses each file that differs, so a match shows that the signed set
is this source. The public key in `keys/` travels with the source.

On an `ai-tools-base` host, the administrator binds the set name to your key
and places the staged set, root-owned, under the local root, as
`ai-tools-assets(5)` states:

```bash
gpg --dearmor -o <set>.gpg keys/<fingerprint>.asc
sudo install -o root -g root -m 0644 <set>.gpg /etc/ai-tools/assets-bindings.d/<set>.gpg
printf 'set=%s\nsigners=[openpgp:%s]\nkeyring=%s\n' \
    <set> <fingerprint> /etc/ai-tools/assets-bindings.d/<set>.gpg \
    | sudo tee /etc/ai-tools/assets-bindings.d/<set>.conf
sudo mkdir -p /usr/local/share/ai-tools-assets
sudo cp -r build/<set> /usr/local/share/ai-tools-assets/<set>
sudo ai-tools-admin assets enable --set <set>
```

A host without `ai-tools-base` places the skills with
`python3 ../ai-tools-assets-tools/tools/link-set <set> --target <skills directory>`.

A signature proves who signed a set and that its files are the ones signed,
not that their content is safe: the validator refuses the shapes that act
before a person reads an asset, and the reader inspects the rest. A host
withdraws its trust in a publisher by removing the binding; there is no
revocation list or transparency log of what was signed under a set name.

## Layout

```text
publisher.conf                  who publishes; new-publisher writes it
sets/change-me-example/         the example set, one skill; new-publisher replaces it
.claude-plugin/marketplace.json written by sync-manifests
.github/workflows/              validate and release, calling ai-tools-assets-tools
keys/                           the publisher's public key
LICENSE, LICENSES/, REUSE.toml  MIT-0 for the template
```

## Licence

The template is [MIT-0](LICENSE): copy it, and replace the notice with your
own, without attribution; `new-publisher` writes your licence, MIT unless
`--license` names another. The authors are the committers recorded in this
repository's history, each commit signed off under the DCO.
