---
	author: "Lahfa Samy"
---


# Motivation for this project

MetaCall is packaged for Linux and for Windows, it was not packaged for MacOS but some efforts were already being made so that it can be packaged for MacOS.

I've gotten to package MetaCall for NixOS and for the ArchLinux User Repositories, namely for ArchLinux.

However, more people were asking for Metacall to be packaged for MacOS, than for Ubuntu, Debian and others Linux distributions.

Metacall's packaging is needed for MacOS and Linux in order to allow easier interoperability of multiples programming langages (making Python interacting NodeJS for instance).

Hence I started working on a proposal to package MetaCall natively for MacOS using CMake but I had no MacOS machine, so I had to do some preparation.

# Packaging for MacOSX

## Setting up QEMU OSX-KVM

First of all, I had to setup an [OSX-KVM virtual machine with QEMU](https://github.com/kholia/OSX-KVM), it was necessary in order to have some kind of MacOS kind of machine.

The setup was optimized using these [instructions](https://github.com/sickcodes/osx-optimizer) and required about 15/20GB of free space (recommended is 40/50GB for building NodeJS support), I thus installed MacOS BigSur and got to setup `ssh` on it as the GUI was pretty slow to interact with, even with every tweaking possible, if one has a Thunderbolt 3/4 port, they could IOMMU passthrough the GPU to the QEMU-OSX VM and have a faster GUI, but I didn't have this choice.

Once this was done, I would start looking at the Windows script for building MetaCall and try and copy its steps and write one for MacOS in Bash.

## Writing a build script in bash

I started by looking at the steps the metacall/windows-distrubutable and started making bash functions to have a modular script and for debugging more easily.

Thus, I began working on MetaCall Python support, I tried installing manually Python, then building MetaCall, but then I had to make sure I used an contained Python package, such as the pkg file given by the Python Foundation using the `install` command on MacOS, that allows installing software on any MacOS system.

I've hit a first quirk of working with MacOS, that is that the `install` command does not really take into account from time to time the `-target` path given as a parameter to install the Python, it just updates the system Python install.

Then, I tried to just have a binary isolated Python into a folder and try to make use of this one, instead of the system one.

To do so, I followed the instruction of this blog, but I hit an error due to Apple signing all their compiled binaries.

I disabled the code signing and retried again, and ran into a weirder error, trying to trace system calls, led me to a bug in Dtrace (strace equivalent on Linux) Apple's closed source implementation.

## First big hurdle, Apple isn't too friendly to open source packaging in VMs

The first big hurdle, I'd hit has been that Apple codesigns all their binaries.

In order to package MetaCall, more precisely adding the Python support to MetaCall, a portable Python was needed.

I started searching if I could make the system Python on MacOS portable, [thus I followed these instructions](https://joaoventura.net/blog/2016/embeddable-python-osx/) however the last step failed, due to the codesigning being invalidated by using `otool` to change the `rpath` (runtime Path) of the Python binary in order to make it relocatable.

I then tried to disable codesigning with `codesign --remove-signature` but the execution kept failing with an even more obscure error about a `posix_spawn` syscall, failing to end properly, [this StackExchange answer notes indeed that removing the signature, can lead to malformed executable, which may be an explanation why, this weird error popped up out of nowhere](https://reverseengineering.stackexchange.com/a/13623/34891).

I tried to trace syscalls using `dtruss`, an `strace` equivalent on MacOS, that is proprietary developed by Apple, however this led me to discover how buggy is the syscall tracing developed by Apple.

In the end, we concluded that:

- Patching SMBIOS values in the OSX-QEMU VM to avoid AppleID anti-VM bans in order to connect an Apple Developper account and codesign binaries after having modified them.
- a MacOS device with an Apple Developper account.
- Ask [MacStadium](https://www.macstadium.com/), an open source licence, to work with real Macs but remotely.
- Use relocatables binaries, we choose this solution as it was the fastest one, we could go for in order to achieve the project in due deadlines.

More details are found, in this [issue upstream](https://github.com/metacall/distributable-macos/issues/4).

## Moving on to Homebrew packaging

### Packaging multiples langages

#### Compiler bug for Ruby support

I ran into an LLVM compiler [bug in AppleClang that was fixed by introducing a new flag that fixes the compilation for Ruby](https://github.com/nginx/unit/issues/653#issuecomment-1063524279), namely `-fdeclspec`, see this [pull request commit](https://github.com/metacall/core/pull/295/commits/90189338e597d66421183e3dab22f14c62b0a7d2).

I had to add this flag in a CMakeLists file in order to fix building Ruby support for MacOS using Homebrew, [on MacOS Monterey, the build was still failing, I needed to add the flag into another CMakeList file](https://github.com/metacall/core/pull/310), despite it working, on older MacOS versions it was working.

Setting up GitHub Actions for MacOS (10.15 - Catalina, 11 BigSur, 12 Monterey), in order to do these tests greatly helped me work faster on the developpement and helping prevent bugs from getting upstream silently.

#### NodeJS support quirks

NodeJS support was next after Ruby and Python was working properly for the brew building script.

NodeJS was a bit harder to build than the two previous langage, as `libnode` was needed and the v8 Javascript needed to be compiled everytime, taking about 30 minutes to an hour of compilation on my 8 threads, Ryzen 7 3700U laptop.

I began using `node@14`, since this was tested and recommended by the maintainers of MetaCall as a known working version of Node.

The build worked alright locally, and the testing went fine as well, TypeScript's test had to be tweaked with a bit of help from maintainers of MetaCall.

Homebrew *assertions* more precisely `assert_match` was used in the `test` Ruby block in order to achieve reliable tests locally and for the CI/CD.

In order for Node to build in the brew CI/CD and so that the tests could pass.

I needed to have a new option flag `-DNodeJS_INSTALL_PREFIX=/tmp/#{version}`, allowing a custom path where Node would get built, as **permission were denied on the CI/CD Homebrew Core environnement**.

### Merging upstream Homebrew

I had to solve many warnings of brew, most of them were simple to solve, some were about formatting, some about making sure I specified the canonical names of dependencies, some of them were more trickier, one of them was about some libraries being installed in /lib instead of /libexec and this would need a heavy change made by maintainers, thus the PR Upstream to Homebrew wasn't merged yet.

## Packaging natively

### Python relocatable

[An issue was opened before we switched to packaging with Homebrew](https://github.com/metacall/distributable-macos/issues/4) in order to track what would be done later, to implement native packaging, maintainers of MetaCall had been looking for solutions to the issue, we had ran into, namely relocatable binaries for MacOS for Python, Ruby and others langages if needed.

#### Patching CMakeList file

#### Fixing the rpath

The runtime path had to be tweaked in order to account for the path of the Python relocatable files and libraries.

Thus the [patch_cmake_python function in bash was written](https://github.com/metacall/distributable-macos/commit/07f8eb43a53f9499b748346117e26109a6aafa35#diff-4d2a8eefdf2a9783512a35da4dc7676a66404b6f3826a8af9aad038722da6823R105-R114).

### NodeJS support

#### SDK, CC, CCLANG, xcrun

[Node support was failing in a quite weird way at first](https://github.com/metacall/distributable-macos/issues/6), building natively with Node support, failed due to some basic C headers (such as `string.h`, `errno.h`,etc.) who were apparently missing.

MetaCall maintainers investigated this issue as it was quite weird, and came up with a few ENV variables that fixed this issue.

#### CMake not finding the built NodeJS libraries

Once, the above issue, was solved, CI/CD ran into another issue, CMake was failing to locate the libraries or just detect that NodeJS was indeed built.

I added debugging messages in `cmake/FindNodeJs.cmake` before the failure, in order to check if CMake was trying to use the NodeJS preinstalled in the CI/CD GitHub Actions environnement or the one that the building script was compiling.

Along with MetaCall's maintainers, we find out that the NodeJS that was preinstalled in the CI/CD was being detected and used instead of the one, that was being built.

A more thorough investigation of the NodeJS CMake detection code has to be done and will be done in order to fix this issue.

## Further work

- Merge the Ruby [`metacall.rb`](https://github.com/AkechiShiro/homebrew/blob/main/metacall.rb) brew building script into [Homebrew/core](https://github.com/Homebrew/homebrew-core) repository, by fixing the last warning left.
- A solution should be found just like Python relocatable but for Ruby, as no solution was found and thus native packaging for MacOS Ruby support is not yet supported.
- DotNet support for Homebrew package.
- Homebrew ruby building script should be PR after maintainers have fixed the last warning about libraries not being in libexec.
- [Solving this issue](https://github.com/metacall/distributable-macos/issues/5), forking of the [repo of Python relocatable](https://github.com/gregneagle/relocatable-python) and merging at least 2 PRs that can help in making relocatable Python better and more robust.
