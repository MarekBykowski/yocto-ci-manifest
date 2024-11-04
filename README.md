## yocto-CI manifest

Yocto-CI manifest is a build-up of the mandatory git repositories, listed below, enabling running `testimage` tests agains QEMU or SIMICS. The manifest comprises the following repositories:
- meta-cxl 
- meta-openembedded
- poky

## Table-Of-Contents

- [Set up working environment](#set-up-working-environment)  

## Set up access to private repos

First, all/almost all of the repos I keep our Yocto are `private`, aka nobody without explicitly graneded access cannot access, even see it. To be able to access it auto from within your account, namely without giving the user/password each time, you have to cache your credentials. One method I explored successfully is to generate a personal access token (classic) and cache it via github CLI (`gh`). Go with

- go to https://github.com/settings/tokens and click on `generate a personal access token (classic)`, figure below

![image](https://github.com/user-attachments/assets/44e5a5d7-bb84-4de8-9587-c98ee95f4931)

- then set `no expiration` and click on `repo` giving the full control over the private repos, as shown in the figure as fallows

![image](https://github.com/user-attachments/assets/f4dd2a8d-b8af-4886-852a-394db8f3e881)

## hhhh

Clone all the required repos using the `git-repo` tools reading the git menifest file.

```
git clone https://gerrit.googlesource.com/git-repo
./git-repo/repo init -u https://github.com/MarekBykowski/yocto-ci-manifest.git
./git-repo/repo sync
```
