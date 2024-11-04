## yocto-CI manifest

Yocto-CI manifest is a build-up of the mandatory git repositories, listed below, enabling running `testimage` tests agains QEMU or SIMICS. The manifest comprises the following repositories:
- meta-cxl 
- meta-openembedded
- poky

## Table-Of-Contents

- [Set up Simics/vp/cosim from scratch](#set-up-simicsvpcosim-env-from-scratch)  
- [Note for building for vp](#note-for-building-for-vp)
- [Email](#email)

Clone all the required repos using the `git-repo` tools reading the git menifest file.

```
git clone https://gerrit.googlesource.com/git-repo
./git-repo/repo init -u https://github.com/MarekBykowski/yocto-ci-manifest.git
./git-repo/repo sync
```
