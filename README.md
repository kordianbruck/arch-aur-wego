# arch-aur-wego

Packaging sources for the [`wego`](https://aur.archlinux.org/packages/wego) AUR package, a weather
client for the terminal ([upstream](https://github.com/schachmat/wego)).

This GitHub repository is the source of truth. The AUR repository is only a release target and is
updated automatically, so do not push to it by hand.

## Workflow

1. Open a pull request against `master` on
   [kordianbruck/arch-aur-wego](https://github.com/kordianbruck/arch-aur-wego).
2. CI ([`.github/workflows/aur.yml`](.github/workflows/aur.yml)) builds the package in an Arch Linux
   container, verifies that `.SRCINFO` matches the `PKGBUILD`, runs `namcap` and smoke-tests the
   installed binary.
3. Merge the PR. Merging alone does **not** publish anything.
4. Push a release tag named `<pkgver>-<pkgrel>` (e.g. `2.4-1`) that points at a commit on `master`.
   CI builds that commit again, checks that the tag matches `.SRCINFO` and pushes a snapshot of the
   package files to `ssh://aur@aur.archlinux.org/wego.git` as one commit.

The AUR commit message is the message of the annotated tag, or `Update to <tag>` for a lightweight
tag. The commit author is the author of the tagged commit. Unrelated commits on `master` (CI tweaks,
Dependabot bumps, docs) never reach the AUR and never supply its commit message.

Only files not marked `export-ignore` in [`.gitattributes`](.gitattributes) are published. The AUR
rejects subdirectories, so every repo-only file or folder must be listed there.

## Updating to a new upstream release

```sh
git switch -c update-X.Y
# bump pkgver, reset pkgrel=1
updpkgsums                          # from pacman-contrib
makepkg --cleanbuild                # test build
makepkg --printsrcinfo > .SRCINFO   # CI fails if this is stale
git commit -am "Update to X.Y"
git push -u origin update-X.Y       # then open a PR and merge it

# after the merge
git switch master && git pull
git tag -a X.Y-1 -m "Update to X.Y"
git push origin X.Y-1               # publishes to the AUR
```

For packaging-only changes, bump `pkgrel` instead of `pkgver` and tag `X.Y-<pkgrel>`.

## One-time setup

Remotes for a local clone:

```sh
git clone git@github.com:kordianbruck/arch-aur-wego.git
cd arch-aur-wego
git remote add aur ssh://aur@aur.archlinux.org/wego.git   # read-only use, e.g. to compare
```

To let CI publish to the AUR:

1. Generate a dedicated key with `ssh-keygen -t ed25519 -f aur_ci -C "arch-aur-wego CI"`.
2. Add `aur_ci.pub` to the AUR account that maintains `wego` (*My Account → SSH Public Key*).
3. Store the contents of `aur_ci` as the repository secret `AUR_SSH_PRIVATE_KEY`
   (`gh secret set AUR_SSH_PRIVATE_KEY < aur_ci`), then delete the local private key.
4. Protect `master` so that changes only land through PRs with a passing `build` check.
5. Optionally add a tag ruleset for `*-*` so only maintainers can create or move release tags.
