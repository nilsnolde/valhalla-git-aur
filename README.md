# `valhalla-git` PKGBUILD

## Publishing process

The package is fully defined by the PKGBUILD & .SRCINFO files.

There is a daily-running GHA which:

- checks whether we even need an update (i.e. is the tip of `master` different from `_git_commit` variable?)
- updates the PKGBUILD's `_git_commmit` variable which will update the `source` and set the `VALHALLA_VERSION_MODIFIER`
- updates the .SRCINFO's `pkgver` variable with the new version
- pushes the changes to this repository
- finally publishes the package on the AUR git remote

> [!WARNING]
> If anything else other than `pkgver` needs to be updated in PKGBUILD, the git update has to be done manually:

```
# Make your changes to PKGBUILD
makepkg --printsrcinfo > .SRCINFO
git commit -am "release <commit>"
git push
# Optionally update the AUR remote, GHA will also do that on the next scheduled run
```

## Initial packaging

More a log of what I did to make the first packaging attempt work:

1. [Building a clean chroot](https://wiki.archlinux.org/title/DeveloperWiki:Building_in_a_clean_chroot#Setting_up_a_chroot)
2. Install the AUR dependencies to the system
    ```
    yay -Syu prime_server unordered_dense
    ```
3. Build the package in CHROOT with the AUR packages

    ```
    CHROOT=~/chroot
    cd ~/dev/pacman/valhalla-git
    makechrootpkg -r $CHROOT \
      -I ~/.cache/yay/prime_server/prime_server-0.7.0-2-x86_64.pkg.tar.zst \
      -I ~/.cache/yay/czmq-git/czmq-git-1:4.2.1.r150.g5b5c6402-2-x86_64.pkg.tar.zst \
      -I ~/.cache/yay/unordered_dense-git/unordered_dense-git-r227.73f3cbb-2-x86_64.pkg.tar.zst
    ```

    This will build the tar'd compressed package to the current dir. 

    To interact with the source tree/chroot, the path is e.g. `~/chroot/nilsnolde/build/valhalla-git/src/valhalla-git`. 
4. Install the package with `sudo pacman -U <path_to.tar.zst>`

## TODO

  - [ ] Create the AUR git repo remotely
  - [ ] automation for publishing AUR package, e.g. https://jabeztho.com/blog/automating-aur-package-maintenance-with-github-actions/
  - [ ] don't forget to replace with `sed`: 
    - `pkgver=printf "%s" "$(git describe --long --tags --abbrev=7 | sed 's/\([^-]*-\)g/r\1/;s/-/./g;s/v//g')"`
    - `git_commit=git rev-parse --short HEAD`
  - [ ] run smth like `run-python_valhalla` in `check()`: needs appropriate `checkdepends` packages like spatialite-tools, unzip etc
  - [ ] ideally add a GHA step which builds the PKGBUILD in a chroot environment and runs `nampac` (caution: also with errors it's exiting with 0)
