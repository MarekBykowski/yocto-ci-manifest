## yocto-CI manifest

Yocto-CI manifest is a build-up of the mandatory git repositories, listed below, enabling running `testimage` tests https://docs.yoctoproject.org/dev/dev-manual/runtime-testing.html#running-tests against `QEMU` or `Simics`. The manifest comprises the following repositories:
- meta-cxl https://github.com/MarekBykowski/meta-cxl.git
- meta-openembedded
- poky https://github.com/MarekBykowski/poky.git

## Table-Of-Contents

- [Set up access to private repos](#set-up-access-to-private-repos)
- [Where do we work on?](#where-do-we-work-on)
- [Work with the repos](#work-with-the-repos)
  -  [Clone the repos](#clone-the-repos)
  -  [Update the repos](#update-the-repos)
- [Run Yocto to produce the images](#run-yocto-to-produce-the-images)
- [Switch from `fatal` to `non-fatal` tests](#switch-from-fatal-to-non-fatal-tests)
- [`Yocto-CI` `testimage` testing](#run-yocto-ci-testimage-testing)
- [`Yocto-CI` `oe-test` testing](#yocto-ci-oe-test-testing)

## Set up access to private repos

First, all/almost all of the repos I keep our Yocto are `private`, aka nobody without explicitly graneded access can access, neither see it. To be able to access it auto from within your account, namely without giving the user/password each time when accessing you have to cache the git credentials. One method I explored successfully is to generate a personal access token (classic) and cache it with `github CLI gh`

- go to https://github.com/settings/tokens and click on `generate new token` (right hand side)

<p align="center">
  <img width="700" src="https://github.com/user-attachments/assets/2692acc5-a204-44eb-ac6b-b1baa8dda923">  
</p>

- then in the input label `Note` write `yocto-ci`, set expiration to `no expiration` and click on `repo` giving the full control over the private repos, as shown in the figure as follows

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

**This is important!** I have prepared a special path `/yocto/yocto-team` on `GNR` using the `LVM` (logical-volume-manager) concatenating the free spaces across various drives in which all of us should create a `user` dir and work from there.
If not abiding by to it we will exhaust the space on GNR shortly. Never ever try these steps in your home dir. Producing Yocto images and this is what we are going to do is ~100G occupation. Running tests on top is almost no extra space but the space the python test scripts take.

```
mkdir -p /yocto/yocto-team/$USER
cd /yocto/yocto-team/$USER
```

## Work with the repos

### Clone the repos

Clone all the repos necessary using the `git-repo` tool and the manifest file `default.xml` from within this repo.

```
git clone https://gerrit.googlesource.com/git-repo
./git-repo/repo init -u https://github.com/MarekBykowski/yocto-ci-manifest.git
./git-repo/repo sync
```

### Update the repos

```
cd /yocto/yocto-team/$USER
./git-repo/repo sync
```

## Run `Yocto` to produce the images

Go to the `poky` distro and source for the machine under test, `QEMU` or `Simics`. `QEMU` is labaled `cxl`, `Simics` is `cxl-simics`
```
cd /yocto/yocto-team/$USER/poky/
source oe-init-build-env <cxl | cxl-simics>
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

## Switch from fatal to non-fatal tests

By default `Yocto-CI` does `fatal` tests in which if in a set of tests any throughout fails the run is interrupted and the remaining tests after the first failed are not run, eg. if we have four tests

1. test ok
2. test ok
3. test nok
4. test ok

tests #1 thr #2 are run, test #3 fails and test #4 is not run even though it could result with success.

I have written a facility in which I changed the behaviour from `fatal` to `non-fatal`, in which all the tests run presenting us with the result status.

## `Yocto-CI` `testimage` tests

Producing Yocto artifacts (steps above) is a prerequisite and a one-shot action for running `testimage` tests. By saying that I mean there is no need to re-generate the images each time you run the tests, even when you add/modify the tests there is no need to re-generate the rootfs. Typically the build master should let you know when you should update for the rootfs.

Each time you log out and then in to `GNR` you have to switch to the `poky` distro and source for the machine under test:
```
cd /yocto/yocto-team/$USER/poky/
source oe-init-build-env <cxl | cxl-simics>
```

Before running the tests make sure the rootfs image is updated and built (`bitbake core-image-cxl-sdk` shall run to completion and without any errors). Then set for the Yocto-CI parameters in `conf/local.conf`. Mine are/were at the time of writing the article:
```
TEST_TARGET_IP = "GNR-JF04-5350.jf.intel.com:2222"           
TEST_SUITES = "ping ssh cpdk"
```

- `TEST_TARGET_IP` defines the IP address of the Linux machine the `QEMU`/`Simics` runs on and the port number the these are available at. For `QEMU` as we typically run it from within `GNR` the IP is `GNR-JF04-5350.jf.intel.com`. The port must be checked in the log files of the `QEMU`. `Simics` are typically run on the Intel servers, so you need to check the IP for the port this wiki may be of help https://github.com/MarekBykowski/avery_qemu/wiki/Simics
- `TEST_SUITES` defines the tests to run

From within your working directory `/yocto/yocto-team/$USER/poky/<cxl|cxl-simics>` the tests pre-defined (written by Yocto folks) are here `../meta/lib/oeqa/runtime/cases`. Tests that we write should go to our meta layer `meta-cxl` to `../../meta-cxl/lib/oeqa/runtime/cases`.

Kick off the `QEMU` or `Simics` testing (https://github.com/MarekBykowski/avery_qemu/wiki/B2B-with-Yocto-qcow). After the machine comes up check for the ssh port the `QEMU` or `Simics` listens to, and set the port and IP address of the `QEMU` or `Simics`to `conf/local.conf:TEST_TARGET_IP`. Afterwards run the tests with:

### Run `Yocto-CI` `testimage` testing

```
bitbake core-image-cxl-sdk -c testimage
```

The command will run all the tests defined in `TEST_SUITES`

###  Where the `Yocto-CI` `testimage` test results are?

If you are not yet in the Yocto-CI env. go there and source for the machine under test:
```
cd /yocto/yocto-team/$USER/poky/
source oe-init-build-env <cxl | cxl-simics>
```

Then look for the logfiles in `tmp/log/oeqa`

## `Yocto-CI` `oe-test` testing

`oe-test` tool is a shell-like `testimage` tool. It enables you to list and run all the tests, a subset of tests or a single test and inspect the logfiles.

Go to poky and source for the `machine` under test, either for `QEMU` or `Simics`.

```
cd /yocto/yocto-team/$USER/poky/
source oe-init-build-env <cxl | cxl-simics>
```

### List tests

List available modules from `<dir>`
```
oe-test runtime --list-tests module /yocto/yocto/meta-cxl/lib/oeqa/runtime/cases
```

List available classes from `<dir>`
```
oe-test runtime --list-tests class /yocto/yocto/meta-cxl/lib/oeqa/runtime/cases 
```

List available tests from `<dir>`
```
oe-test runtime --list-tests name /yocto/yocto/meta-cxl/lib/oeqa/runtime/cases  
```

List available tests from `<dir1>` and `<dir2>`
```
oe-test runtime --list-tests name /yocto/yocto/meta-cxl/lib/oeqa/runtime/cases /yocto/yocto/meta-cxl/lib/oeqa/runtime/cases
```

### Run tests

Run a tests group named `demo1`
```
oe-test runtime --target-type simpleremote --target-ip 127.0.0.1:2223 --server-ip 127.0.0.1 \
--packages-manifest data/manifest --test-data-file data/testdata.json /yocto/yocto/meta-cxl/lib/oeqa/runtime/cases \
--run-tests demo1
```

Run a single test `demo2.DEMO2Test.test_script_fail`
```
oe-test runtime --target-type simpleremote --target-ip 127.0.0.1:2223 --server-ip 127.0.0.1 \
--packages-manifest data/manifest --test-data-file data/testdata.json /yocto/yocto/meta-cxl/lib/oeqa/runtime/cases \
--run-tests demo2.DEMO2Test.test_script_fail
```

Run a list of tests putting one after another
```
oe-test runtime --target-type simpleremote --target-ip 127.0.0.1:2223 --server-ip 127.0.0.1 \
--packages-manifest data/manifest --test-data-file data/testdata.json /yocto/yocto/meta-cxl/lib/oeqa/runtime/cases --run-tests \
demo2.DEMO2Test.test_script_fail \
demo2.DEMO2Test.test_method_sshd
```

### Where the tests results are?

The logfiles are located in `/yocto/yocto-team/$USER/poky/<cxl | cxl-simics>/data`
