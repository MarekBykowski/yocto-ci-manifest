## yocto-CI manifest

Yocto-CI manifest is a build-up of the mandatory git repositories, listed below, enabling running `testimage` tests agains QEMU or SIMICS. The manifest comprises the following repositories:
- environment https://github.com/intel-sandbox/cosim-env-setup.git
- cxl_relay https://github.com/intel-sandbox/cxl_relay.git
- vp https://github.com/intel-restricted/applications.simulators.isim.vp.git 

```
git clone https://gerrit.googlesource.com/git-repo
./git-repo/repo init -u https://github.com/MarekBykowski/yocto-ci-manifest.git
./git-repo/repo sync
```
