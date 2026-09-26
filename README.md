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
3. Once the PR is merged, the same workflow runs on `master` and pushes it to
   `ssh://aur@aur.archlinux.org/wego.git`, which publishes the release on the AUR.

The AUR only accepts fast-forward pushes to `master`, so never rewrite `master` history here.

## Updating to a new upstream release

```sh
git switch -c update-X.Y
# bump pkgver, reset pkgrel=1
updpkgsums                          # from pacman-contrib
makepkg --cleanbuild                # test build
makepkg --printsrcinfo > .SRCINFO   # CI fails if this is stale
git commit -am "Update to X.Y"
git push -u origin update-X.Y       # then open a PR
```

For packaging-only changes, bump `pkgrel` instead of `pkgver`.

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
