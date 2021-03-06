# Building Swift on ARM

A few, very simple, bash scripts to clone, configure and build Swift on ARM devices. 

Derived from [package-swift](https://github.com/iachievedit/package-swift) by [@iachievedit](https://twitter.com/iachievedit), some patches from [swift-arm](https://github.com/swift-arm/) by [@hpux735](https://twitter.com/hpux735).

Currently this project builds Swift 3.1.1 on Ubuntu Mate 16.04.x (get it [here](https://www.dropbox.com/s/w0fxwqx7t1llixx/swift-3.1.1-RPi23-1604.tgz?dl=0)), Raspbian/Stretch (get it [here](https://www.dropbox.com/s/v6oslfta6u773rj/swift-3.1.1-RPi23-RaspbianStretchAug17.tgz?dl=0) for ARMv7 or [here](https://www.dropbox.com/s/rnpcenhk48piskd/swift-3.1.1-Rpi1armv6-RaspbianStretchAug17.tgz?dl=0) for ARMv6) and usually on other Debian-based distributions as well. SPM compiles correctly for both but the REPL is, as usual, not available.

Compiling 3.1.1 on Raspbian Stretch requires an additional libdispatch patch available under `swift-corelibs-libdispatch.diff/raspbian/`, apply it with `patch -p1 < ../swift-corelibs-libdispatch.diffs/raspbian/c01599944879e58556a6e34585ac6539f5e83000_no_membarrier.diff` from the libdispatch directory.

For the latest updates on Swift on ARM, check out my blog [here](https://www.uraimo.com/category/raspberry/).

Check out Helge Heß's project [dockSwiftOnARM](https://github.com/helje5/dockSwiftOnARM) to build Swift in a Docker container or to [build a cross-compiling toolchain](https://github.com/helje5/dockSwiftOnARM/blob/master/toolchain/README.md) that will allow you to build arm binaries directly from your Mac using a precompiled swiftc for ARM.


The scripts:

- clone.sh - Install dependencies and clones the main Swift repository and all the related projects

- checkoutRelease.sh - Resets all repos, updates them, checks out a specific tag (3.1.1 at the moment) and apply the patches

- build.sh - Build

- clean.sh - Clean all build artifacts 


## Building instructions

First of all, use a suitably sized sd-card, at least 16Gb in size.

Configure a swap file of at least 2Gb, on Ubuntu:

    sudo fallocate -l 2G swapfile
    sudo chmod 600 swapfile
    sudo mkswap swapfile
    sudo swapon swapfile
    
You'll need to manually enable the swap file with `swapon` *each time you reboot* the RaspberryPi (or the system will just run without swap).

On Raspbian, open `/etc/dphys-swapfile` and edit:

    CONF_SWAPSIZE=2048
    
Save the file and:

    sudo /etc/init.d/dphys-swapfile stop
    sudo /etc/init.d/dphys-swapfile start
    
Now, call the included scripts as follows:

1. Launch `clone.sh` that will install the required dependencies (_git cmake ninja-build clang-3.8 python uuid-dev libicu-dev icu-devtools libbsd-dev libedit-dev libxml2-dev libsqlite3-dev swig libpython-dev libncurses5-dev pkg-config libblocksruntime-dev libcurl4-openssl-dev autoconf libtool systemtap-sdt-dev_), fix clang links and clone apple/swift with all its dependecies.

2. Run `checkoutRelease.sh` that will select the current release (3.1.1) and apply the needed patches. These patches cover the basic Raspi2/3 with Xenial case, but even I've had many report of successful build onn different setups, additional patches  could still be needed on different boards/OSs. I recommend to just try and if you get an error, verify if one of the patches in the subdirectories of the <failing_component>.diffs can solve your issue. To apply it manually going in the `<failing_component>` directory and run `patch -p1 < ../<failing_component>.diffs/otherdebians/fix.diff`.

3. Once done, start the build with `build.sh`.

4. Once the build completes a few hours later, you'll have a `swift-3.1.1.tgz` archive containing the whole Swift compiler distribution. Once decompressed you'll find the Swift binaries under `usr/bin`.

I recommend to perform all these operations in a permanent background `tmux` or `screen` session (`CTRL+B d` to detach from the session and `tmux a` to reattach to it when you ssh again into the RaspberryPi).

Additional steps could be required in some cases (on a RaspberryPi 1 or for Raspbian) [check the latest ARM posts on my blog for additional info](https://www.uraimo.com/category/raspberry/).

If you plan to build 4.0+ and if your cmake is older than 3.6.x (Ubuntu Mate ships with 3.5.2), you'll also need to [download](https://cmake.org/files/) and compile from sources a newer release of cmake, I recommend 3.7.2.

To build a different release than the one currently configured in the script, open `checkoutRelease.sh` and `build.sh` and modify the variables on top, with the branch name for the release and the release name for the tgz respectively.
