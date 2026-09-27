# AGENTS.md

Packaging sources for the [`wego`](https://aur.archlinux.org/packages/wego) AUR package. This
GitHub repository is the source of truth; the AUR git repo is a publish target that CI updates. See
[README.md](README.md) for the human-facing version of this.

## Layout

- `PKGBUILD`, `.SRCINFO`: the package. These (plus any future top-level patch or `.install` files)
  are what the AUR receives.
- `.github/workflows/aur.yml`: `build` job (PRs, `master`, tags) and `publish` job (release tags only).
- `.gitattributes`: `export-ignore` list of repo-only files that must not reach the AUR.
- `.claude/skills/aur-release/`: step-by-step skill for updating and releasing the package.

## Rules

- Never push to `ssh://aur@aur.archlinux.org/wego.git` by hand. Publishing happens only when a
  `<pkgver>-<pkgrel>` tag (e.g. `2.4-1`) is pushed for a commit on `master`.
- Always regenerate `.SRCINFO` after editing `PKGBUILD` (`makepkg --printsrcinfo > .SRCINFO`); CI
  fails otherwise.
- Checksums come from `updpkgsums`, never typed by hand.
- New upstream version: bump `pkgver`, reset `pkgrel=1`. Packaging-only change: bump `pkgrel`.
- The AUR rejects subdirectories. Any new repo-only file or folder must get an `export-ignore`
  line in `.gitattributes`; new package files must live at the top level.
- The AUR commit message comes from the release tag (annotated tag message, or `Update to <tag>`),
  not from the latest `master` commit, so write a meaningful tag message.
- Develop on a branch and go through a PR; do not create or push release tags unless asked.

## Checks

CI runs everything in an `archlinux:base-devel` container. On an Arch machine the equivalent is:

```sh
makepkg --printsrcinfo | diff -u .SRCINFO -
makepkg --cleanbuild
namcap PKGBUILD *.pkg.tar.*
```

`git archive HEAD | tar -t` shows exactly which files would be published.
