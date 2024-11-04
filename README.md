## yocto-CI manifest

Yocto-CI manifest is a build-up of the mandatory git repositories, listed below, enabling running `testimage` tests agains `QEMU` or `Simics`. The manifest comprises the following repositories:
- meta-cxl 
- meta-openembedded
- poky

## Table-Of-Contents

- [Set up access to private repos](#set-up-access-to-private-repos)
- [Where do we work on?](#where-do-we-work-on)
- [Clone all the repos required](#clone-all-the-repos-required)
- [Run Yocto to produce the images](#run-yocto-to-produce-the-images)
- [Run `Yocto-CI` `testimage` tests](#run-yocto-ci-testimage-tests)
- [`Yocto-CI` `testimage` test results](#yocto-ci-testimage-test-results)

## Set up access to private repos

First, all/almost all of the repos I keep our Yocto are `private`, aka nobody without explicitly graneded access can access, neither see it. To be able to access it auto from within your account, namely without giving the user/password each time when accessing you have to cache the git credentials. One method I explored successfully is to generate a personal access token (classic) and cache it with `github CLI gh`

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

Once you have the token copied run `gh auth login` pasting it in the last line
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

Clone all the repos necessary using the `git-repo` tool and the manifest file from within this repo.

```
git clone https://gerrit.googlesource.com/git-repo
./git-repo/repo init -u https://github.com/MarekBykowski/yocto-ci-manifest.git
./git-repo/repo sync
```

## Run `Yocto` to produce the images

Go to `poky` distro

```
cd /yocto/yocto-team/$USER/poky/
```

and source either for `QEMU` (`QEMU` in Yocto is referred to as `cxl` machine) or `Simics` (`Simics` in Yocto is referred to as `cxl-simics`) depending on what machine you want to build the artifacts for and run the Yocto-CI against.

```
source oe-init-build-env cxl
```

or 

```
source oe-init-build-env cxl-simics
```

Upon sourcing you will be shown with

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

then build the Yocto images

```
bitbake core-image-cxl-sdk
```

Then on top of it you can run the Yocto-CI `testimage` tests.

## Run `Yocto-CI` `testimage` tests

Producing Yocto artifacts (steps above) is a prerequisite and a one-shot action for running `testimage` tests. By saying that I mean there is no need to re-generate the images each time you run, even add in and run the `testimage` tests unless the source code for artifacts gets changed and you want to pull the changes in.

I truly belive once you are here you will only need to run the commands from here for running the tests and adding and running the tests.

Each time you log out and then in to `GNR` you have to switch to the `poky` distro and source for the machine you are interested in

```
cd /yocto/yocto-team/$USER/poky/
```

If you want to test against `QEMU` go with:

```
source oe-init-build-env cxl
```

If you want to test against `Simics` go with:

```
source oe-init-build-env cxl-simics
```

Before running the tests make sure the rootfs image is updated and built (`bitbake core-image-cxl-sdk` shall run to completion and without the errors). After check for Yocto-CI parameters in `conf/local.conf`. Mine are/were when writing here:
```
TEST_TARGET_IP = "GNR-JF04-5350.jf.intel.com:2222"           
TEST_SUITES = "ping ssh cpdk"
```

- `TEST_TARGET_IP` defines the IP address of the Linux machine the `QEMU`/`Simics` runs on and the port number the these are available at. For `QEMU` as we typically run it from within `GNR` the IP is `GNR-JF04-5350.jf.intel.com`. The port must be checked in the log files of the `QEMU`. `Simics` are typically run on the Intel servers, so you need to check the IP for the port this wiki may be of help https://github.com/MarekBykowski/avery_qemu/wiki/Simics
- `TEST_SUITES` defines the tests to run

From within your working directory `/yocto/yocto-team/$USER/poky/cxl` or `/yocto/yocto-team/$USER/poky/cxl-simics` the tests pre-defined (written by Yocto folks) are accessed in `../meta/lib/oeqa/runtime/cases`. Tests that we write should go to our meta layer `meta-cxl` in `../../meta-cxl/lib/oeqa/runtime/cases`.

I strongly advise each of us creates a branch with his/her own tests and we merge it to the main branch when appropriate.

Kick off the `QEMU` or `Simics` (https://github.com/MarekBykowski/avery_qemu/wiki/B2B-with-Yocto-qcow), check the port the `QEMU` or `Simics` listens to for `ssh`, set the IP addr the `QEMU` or `Simics` along with the port in `conf/local.conf:TEST_TARGET_IP`, then start the tests with:

```
bitbake core-image-cxl-sdk -c testimage
```

The command will run all the tests defined in `TEST_SUITES`

## `Yocto-CI` `testimage` test results

If you are not yet in the Yocto-CI env. go with:
```
cd /yocto/yocto-team/$USER/poky/
```

then source for the machine you are looking for, either for `QEMU`

```
source oe-init-build-env cxl
```

or for `Simics`

```
source oe-init-build-env cxl-simics
```

Then look for the logfiles in `tmp/log/oeqa`
