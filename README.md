## yocto-CI manifest

Yocto-CI manifest is a build-up of the mandatory git repositories, listed below, enabling running `testimage` tests agains QEMU or SIMICS. The manifest comprises the following repositories:
- meta-cxl 
- meta-openembedded
- poky

## Table-Of-Contents

- [Set up access to private repos](#set-up-access-to-private-repos)
- [Where do we work on?](#where-do-we-work-on)
- [Clone all the repos required](#clone-all-the-repos-required)
- [Run `Yocto` to produce the images](#run-yocto-to-produce-the-images)

## Set up access to private repos

First, all/almost all of the repos I keep our Yocto are `private`, aka nobody without explicitly graneded access can access, neither see it. To be able to access it auto from within your account, namely without giving user/password each time when accessing you have to generate cache the git credentials. One method I explored successfully is to generate a personal access token (classic) and cache it via `github CLI gh`

- go to https://github.com/settings/tokens and click on `generate new token` (right hand side)

<p align="center">
  <img width="700" src="https://github.com/user-attachments/assets/2692acc5-a204-44eb-ac6b-b1baa8dda923">  
</p>

- then in the imput label `Note` write `yocto-ci`, set expiration to `no expiration` and click on `repo` giving the full control over the private repos, as shown in the figure as fallows

<p align="center">
  <img width="700" src="https://github.com/user-attachments/assets/7aff979a-8a0c-4c9c-9c83-8176cd636bdb">
</p>

- then scroll down and click on `generate token`
- after it the token is presented with you. Make sure to copy it now as you won’t be able to see it again!

<p align="center">
   <img width="700" src="https://github.com/user-attachments/assets/48efcae5-8548-493d-972a-17bc20db7d31">
</p>

Once you have the token copied run `gh auth login` pasting it in the last raw
```
$ gh auth login                                            
? Where do you use GitHub? GitHub.com                                                 
? What is your preferred protocol for Git operations on this host? HTTPS              
? How would you like to authenticate GitHub CLI? Paste an authentication token        
Tip: you can generate a Personal Access Token here https://github.com/settings/tokens 
The minimum required scopes are 'repo', 'read:org', 'workflow'.                       
? Paste your authentication token:                                                    
```

Congrats! Now your credentials are cached.

## Where do we work on?

**This is important!** I have prepared a special path `/yocto/yocto-team` on `GNR` using the `LVM` (logical-volume-manager) concatenating the free spaces across various drives in which all of us should create his/her `user` dir and work from there.
If not abiding by to it we will exhaust the space on GNR shortly. Never ever try these steps in your home dir. Producing Yocto images and this is what we are going to do is ~400M occupation. Running tests on top is almost no extra space but the space the python test scripts take.

```
mkdir -p /yocto/yocto-team/$USER
cd /yocto/yocto-team/$USER
```

## Clone all the repos required

Then clone all the repos required using the `git-repo` tool and the manifest file from within this repo.

```
git clone https://gerrit.googlesource.com/git-repo
./git-repo/repo init -u https://github.com/MarekBykowski/yocto-ci-manifest.git
./git-repo/repo sync
```

## Run `Yocto` to produce the images

```
$ cd /yocto/yocto-team/$USER/poky/ && source oe-init-build-env cxl
```

You will be shown with

```
### Shell environment set up for builds. ###

You can now run 'bitbake <target>'

Common targets are:
    bitbake core-image-cxl-sdk
    # For testimage and tests in TEST_SUITES run
    bitbake core-image-cxl-sdk -c testimage
    #core-image-cxl-sdk

You can also run generated qemu images with a command like 'runqemu qemux86-64'.

Other commonly useful commands are:
 - 'devtool' and 'recipetool' handle common recipe tasks
 - 'bitbake-layers' handles common layer tasks
 - 'oe-pkgdata-util' handles common target package tasks
```

and as prompted with go with

```
bitbake core-image-cxl-sdk
```

This will produce all the images and top of it you can run the Yocto `testimage` tests.

Notes:
- each time you log out/in to `GNR` you have to source your Yocto/Yocto-CI env., namely
  ```
  cd /yocto/yocto-team/$USER/poky/
  source oe-init-build-env cxl
  ```
- then you can bitbake an `image` or actually any given `recipe`, eg. `bitbake core-image-cxl-sdk`
