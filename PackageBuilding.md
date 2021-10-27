Package Building
================

There are multiple ways to build a package, each with advantages and disadvantages. Which one you choose will depend on your circumstances.

Note: git-ubuntu no longer supports the build argument (neither for source nor for binary builds).


Downloading the orig tarball (optional)
---------------------------------------

If you intend to use more manual methods like `sbuild` or `dpkg-buildpackage` directly, you will probably have to download the orig tarball first.  You can do so by using:

    $ git ubuntu export-orig

It will try to use the `pristine-tar` branch to generate the tarball (and will likely fail), and then it will fallback to downloading the tarball directly from Launchpad.  When it finishes, you should be able to see a link to the orig tarball at `../`.

Building Source Packages
------------------------

### Using dpkg-buildpackage

This method will directly install any dependencies it needs to build, so it's recommended to create an LXD container to do the build. Replace `focal` with the container image you wish to use.

From within the package repository:

    $ lxc launch images:ubuntu/focal builder &&
      sleep 5 &&
      lxc exec builder -- mkdir -p /root/build/package &&
      tar cf - . | lxc exec builder -- tar xf - -C /root/build/package &&
      lxc exec builder -- sh -c 'apt update &&
      apt dist-upgrade -y &&
      apt install -y ubuntu-dev-tools &&
      cd /root/build &&
      pull-debian-source -d $(grep "Source: " package/debian/control | sed "s/Source: \(.*\)/\1/g") $(grep "unstable; urgency=" package/debian/changelog |grep -v ubuntu|head -1|sed "s/.*(\(.*\)).*/\1/g") &&
      cd /root/build/package &&
      apt build-dep -y ./ &&
      dpkg-buildpackage -S' &&
      lxc exec builder -- tar cf - --exclude=package -C /root/build . | tar xf - -C .. &&
    $ lxc delete -f builder

Even though the recommended way to build a source package is to use a pristine environment inside an LXD container, you can also use `dpkg-buildpackage`'s `--no-check-builddeps` option and build the source package locally:

    $ dpkg-buildpackage -S -d


### Signing the Changes File

In order for a source package to be accepted by Launchpad, it must be signed. You can sign it manually with `debsign` (within the package folder):

    debsign ../$(grep "Source: " debian/control | sed "s/Source: \(.*\)/\1/g")_$(grep "urgency=" debian/changelog|head -1|sed "s/.*(\(.*\)).*/\1/g")_source.changes



Building Binary Packages
------------------------

### Using sbuild

Assuming you have configured `sbuild` properly, you can use it to build the binary package:

    $ sbuild

Because of https://bugs.launchpad.net/launchpad/+bug/1699763, it is a good idea to disable the inclusion of `.buildinfo` files in the `*_source.changes` file:

    $ sbuild --debbuildopts='--buildinfo-option=-O'


### As a PPA in Launchpad

Launchpad can build binary packages from your signed source packages and store them in PPA archives. This is especially useful because reviewers of your merge proposal will have a ready binary to test from. The disadvantage is that during busy times, it can take awhile for your package to come up in the build queue.


#### Set the Version String

For the PPA, we need to change the version in the changelog that's lower than the version we plan to release. Since the tilde `~` character sorts lower than everything else in launchpad, we can simply append `~ppa1` to the version string in `debian/changelog`. For example:

    -postfix (3.3.0-1ubuntu0.1) bionic; urgency=medium
    +postfix (3.3.0-1ubuntu0.1~ppa1) bionic; urgency=medium

Note: The command below can be used to modify the version for PPA usage:
sed -i "s/\($(grep "urgency=" debian/changelog|head -1|sed "s/.*(\(.*\)).*/\1/g")\)/\1~ppa1/g" debian/changelog 

Note: If a PPA is used to build the package and the version string was changed like above, once needs to rerun dpkg-buildpackage -S -d.

#### Create the PPA archive

Go to your launchpad page (https://launchpad.net/~your-username) and click "Create a new PPA". Give it a name such that you'll remember what it's about in a few months. A useful form is `package-type-lpbug-description`:

For example:

 * **URL:** postfix-sru-lp1753470-segfault
 * **Display name:** postfix-fix-lp1753470-segfault
 * **Description:** (leave it empty)

Now click "Activate".

It is also helpful to enable all architectures to ensure no build regressions were introduced. Do so by clicking on `Change Details` in the newly created PPA page, and then selecting the other architectures.

#### Upload the source package

    $ dput ppa:kstenerud/postfix-sru-lp1753470-segfault ../bionic-postfix_3.3.0-1ubuntu0.1~ppa1_source.changes

When it finishes, you should be able to see it e.g. https://launchpad.net/~kstenerud/+archive/ubuntu/postfix-postconf-segfault-1753470/+packages

Note: You must wait for the package to build server-side before you can use the PPA to install packages. This might take time depending on how busy things are!
Launchpad also sends status updates notification mails, so monitor your inbox.
