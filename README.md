## yocto-CI manifest

Yocto-CI manifest is a build-up of the mandatory git repositories, listed below, enabling running `testimage` tests agains QEMU or SIMICS. The manifest comprises the following repositories:
- meta-cxl 
- meta-openembedded
- poky

## Table-Of-Contents

- [Set up working environment](#set-up-working-environment)  

## Set up access to private repos

First, all/almost all of the repos I keep our Yocto are `private`, aka nobody without explicitly graneded access can access, neither see it. To be able to access it auto from within your account, namely without giving user/password each time when logging you have to cache your credentials. One method I explored successfully is to generate a personal access token (classic) and cache it via `github CLI gh`

- go to https://github.com/settings/tokens and click on `generate a personal access token (classic)`, figure below

<p align="center">
  <img width="700" src="https://github.com/user-attachments/assets/44e5a5d7-bb84-4de8-9587-c98ee95f4931">  
</p>

- then in `Note` write `yocto-ci`, set expiration to `no expiration` and click on `repo` giving the full control over the private repos, as shown in the figure as fallows

<p align="center">
  <img width="700" src="https://github.com/user-attachments/assets/0dfb35ac-675f-44fa-ae41-672b9fbe1995">
</p>

- then scroll down and click on `generate token`
- make sure to copy the token now as you won’t be able to see it again!

<p align="center">
   <img width="700" src="https://github.com/user-attachments/assets/48efcae5-8548-493d-972a-17bc20db7d31">
</p>
  
## hhhh

Clone all the required repos using the `git-repo` tools reading the git menifest file.

```
git clone https://gerrit.googlesource.com/git-repo
./git-repo/repo init -u https://github.com/MarekBykowski/yocto-ci-manifest.git
./git-repo/repo sync
```
