Running Package Tests
=====================

Packages will have their own tests under `debian/tests`. We need to run those to ensure there are no regressions.

We use autopkgtest to run the tests in a VM or container.


Preparing a Testing Image
-------------------------

You'll need an image to test from. `autopkgtest` will build a suitable image for you. You may want to regenerate the image from time to time to cut down on the number of updates it must run.

The type of image you can use (chroot, container, or VM) depends on the restrictions in `debian/tests/control`(see https://people.debian.org/~mpitt/autopkgtest/README.package-tests.html)

Important restrictions:

 * `breaks-testbed`: This test is liable to break the testbed system. VM or container recommended.
 * `isolation-machine`: You must use a VM to run these tests.
 * `isolation-container`: You must use a VM or container to run these tests.
 * `needs-reboot`: The test reboots the machine, so you must use a VM or container.


#### Building a VM Image

Create an image like this (replacing bionic with your release of choice):

    $ autopkgtest-buildvm-ubuntu-cloud -r bionic -v --cloud-image-url http://cloud-images.ubuntu.com/daily/server

Note: Use `-m` to specify a closer mirror or `-p` to use a local proxy if it's slow.

Copy the resulting image (autopkgtest-bionic-amd64.img) to a common directory like `/var/lib/adt-images`


#### Building a Container Image

    $ autopkgtest-build-lxd ubuntu-daily:bionic/amd64

You should see an autopkgtest image now when you run `lxc image list`.



### Run the Tests

#### In a VM, Manual

Make sure you're one directory up from your package directory and run:

    $ autopkgtest -U -s -o dep8-mypackage mypackage/ -- qemu /var/lib/adt-images/autopkgtest-bionic-amd64.img

Where:

 * `-U`: run apt-get upgrade
 * `-s`: stop and give you a shell if there is a failure. Good to debug
 * `-o dep8-mypackage`: Put your package name in here. Writes output report to the directory dep8-mypackage.
 * `mypackage/`: Put your package name here. The trailing slash tells it to interpret this as a directory rather than a package name.

Everything after the `--` tells it how to run the tests. `qemu` is shorthand for `autopkgtest-virt-qemu`.


#### In a VM, Using the PPA

    $ autopkgtest -U -s -o dep8-mypackage-ppa --setup-commands="sudo add-apt-repository -y -u -s ppa:mylaunchpaduser/bionic-mypackage-fixed-something-1234567" -B mypackage -- qemu /var/lib/adt-images/autopkgtest-bionic-amd64.img

Where (in setup-commands):

 * `-y`: Assume "yes" for all questions
 * `-u`: Run apt-update
 * `-s`: Add the source line as well
 * `-B`: Don't build

Note: In this case, the package name **doesn't** have a trailing slash because we want to install the package.


#### In a Container, Using the PPA

The command only differs after the `--` part. For example:

    $ autopkgtest -U -s -o dep8-mypackage-ppa --setup-commands="sudo add-apt-repository -y -u -s ppa:mylaunchpaduser/bionic-mypackage-fixed-something-1234567" -B mypackage -- lxd autopkgtest/ubuntu/bionic/amd64


### Save the Results

You'll see the tests run:

    autopkgtest [11:47:12]: version 5.3.1
    autopkgtest [11:47:12]: host karl-tp; command line: /usr/bin/autopkgtest -U -s -o dep8-postfix-ppa '--setup-commands=sudo add-apt-repository -y -u -s ppa:kstenerud/postfix-postconf-segfault-1753470' -B postfix -- lxd autopkgtest/ubuntu/bionic/amd64
    autopkgtest [11:47:31]: @@@@@@@@@@@@@@@@@@@@ test bed setup

    ...

    ----------------------------------------------------------------------
    Ran 15 tests in 67.027s

    OK
    autopkgtest [11:49:51]: test postfix: -----------------------]
    autopkgtest [11:49:51]: test postfix:  - - - - - - - - - - results - - - - - - - - - -
    postfix              PASS
    autopkgtest [11:49:52]: @@@@@@@@@@@@@@@@@@@@ summary
    postfix              PASS

Save the last part for the description for your merge proposal.
