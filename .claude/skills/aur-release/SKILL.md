---
name: aur-release
description: Update the wego AUR package to a new upstream version or packaging revision and publish it to the AUR via a release tag. Use when asked to bump wego, release a new pkgrel, or publish to the AUR.
---

# Updating and releasing the wego AUR package

Publishing is tag-driven: CI pushes to the AUR only when a `<pkgver>-<pkgrel>` tag is pushed for a
commit on `master`. The AUR commit message is the annotated tag's message (fallback:
`Update to <tag>`), so the message never depends on whatever landed on `master` last.

## 1. Prepare the change on a branch

1. Find the latest upstream release: `https://github.com/schachmat/wego/tags`.
2. `git switch -c update-<pkgver>` from an up-to-date `master`.
3. Edit `PKGBUILD`:
   - new upstream version: set `pkgver`, reset `pkgrel=1`;
   - packaging-only fix: keep `pkgver`, increment `pkgrel`.
4. Update checksums with `updpkgsums` (pacman-contrib). Without Arch tooling, download the source
   tarball from the `source=` URL and compute `sha512sum`; never guess a checksum.
5. Regenerate metadata: `makepkg --printsrcinfo > .SRCINFO`. Without `makepkg`, edit `.SRCINFO`
   to mirror every `PKGBUILD` change exactly (`pkgver`, `pkgrel`, `source`, `sha512sums`); CI
   diffs it against `makepkg --printsrcinfo` and fails on any mismatch.
6. If possible, `makepkg --cleanbuild` and `namcap PKGBUILD *.pkg.tar.*`.
7. Commit (`Update to <pkgver>` or `Bump pkgrel to <pkgrel>: <reason>`), push, open a PR, and wait
   for the `build` check to pass.

## 2. Release after the PR is merged

Only do this when the user asks for the release.

```sh
git switch master && git pull
git tag -a <pkgver>-<pkgrel> -m "Update to <pkgver>"
git push origin <pkgver>-<pkgrel>
```

The workflow then rebuilds the tagged commit, checks that the tag equals `pkgver-pkgrel` from
`.SRCINFO`, checks that the commit is on `master`, and pushes a snapshot of the non-`export-ignore`
files to the AUR. Watch the `publish` job; if the AUR already matches, it exits without a commit.

## Pitfalls

- Tag name mismatch with `.SRCINFO` fails the job; fix by deleting the tag and re-tagging.
- A new repo-only file (docs, configs, anything in a folder) must be added to `.gitattributes`
  with `export-ignore`, or the AUR push is rejected / pollutes the package. Check with
  `git archive HEAD | tar -t`.
- Never push to the AUR remote directly and never rewrite `master` history.
