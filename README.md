## yocto-CI manifest

Yocto-CI manifest is a build-up of the mandatory git repositories, listed below, enabling running `testimage` tests agains QEMU or SIMICS. The manifest comprises the following repositories:
- meta-cxl 
- meta-openembedded
- poky

## Table-Of-Contents

- [Set up working environment](#set-up-working-environment)  

## Set up access to private repos

First, all/almost all of the repos I keep our Yocto are `private`, aka nobody without explicit graneded access cannot access, even see it. To be able to access it from within your account auto, without giving the user/password each time you have to cache your credentials.
One method I explored successfully is to generate a personal access token (classic) by clicking it on https://github.com/settings/tokens

![image](https://github.com/user-attachments/assets/44e5a5d7-bb84-4de8-9587-c98ee95f4931)


## 

Clone all the required repos using the `git-repo` tools reading the git menifest file.

```
git clone https://gerrit.googlesource.com/git-repo
./git-repo/repo init -u https://github.com/MarekBykowski/yocto-ci-manifest.git
./git-repo/repo sync
```
