# provider-upjet-gcp (Dojo fork)

Fork of [crossplane-contrib/provider-upjet-gcp](https://github.com/crossplane-contrib/provider-upjet-gcp)
carrying patches we need before they land upstream.

`main` mirrors upstream and is not used. We ship from **`release-dojo`**: the
current upstream release tag plus our patches, rebased forward as upstream
releases. Which upstream tag it sits on is recorded in the package
tags. Upstream's own README is kept at [README.upstream.md](README.upstream.md).

(`main` is covered by an org ruleset requiring Wiz status checks, so it cannot
be force-pushed. `release-*` branches are unrestricted and match the build
system's default `RELEASE_BRANCH_FILTER`.)

To be removed when https://github.com/crossplane-contrib/provider-upjet-gcp/pull/1031 is merged.

## Publishing a package

Published artefacts come from **git tags**.

```bash
git tag v3.0.0-dojo.1        # <upstream base>-dojo.<revision>
git push origin release-dojo
git push origin v3.0.0-dojo.1
```

The tag push triggers the *Publish provider packages to GAR* workflow.

**Always verify the push** — the upstream build system makes `publish` a silent
no-op under several conditions, so a green tick is not proof:

```bash
gcloud artifacts docker images list \
  europe-west2-docker.pkg.dev/dojo-creator-platform-nonprod/docker/provider-gcp-cloudtasks \
  --include-tags
```

### Versioning

`v<upstream-base>-dojo.<n>`, e.g. `v3.0.0-dojo.1`.

- states plainly which upstream release it is based on
- rebuilds of the same base increment `<n>`
- a new upstream base restarts at `.1`

Do not use `v3.0.1-...`; that implies an upstream release we are not based on.

## Rebasing onto a new upstream release

```bash
git remote add upstream https://github.com/crossplane-contrib/provider-upjet-gcp.git
git fetch upstream --tags

git checkout release-dojo
git rebase --onto v3.1.0 v3.0.0          # old base -> new base
make generate                            # reconcile generated files
make reviewable
git push --force-with-lease origin release-dojo

git tag v3.1.0-dojo.1
git push origin v3.1.0-dojo.1
```

Conflicts land almost exclusively in generated files. Do not hand-merge them:
take the new base's version (`git checkout --ours <file>`), finish the rebase,
then run `make generate` and amend.

When a patch lands upstream, drop that commit from the branch. When no patches
remain, delete the fork.
