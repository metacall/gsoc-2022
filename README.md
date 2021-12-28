# Google Summer of Code 2022
List of project ideas for students applying to the Google Summer of Code program in 2022 (GSoC 2022).

## Timeline/milestones

The program has not been announced yet but [it is coming with major changes](https://opensource.googleblog.com/2021/11/expanding-google-summer-of-code-in-2022.html).

Please always refer to the [official timeline](https://developers.google.com/open-source/gsoc/timeline). 
  
  
## Application Process

#### 0. Get familiar with GSoC

First of all, and if you have not done that yet, read [the student guide](https://google.github.io/gsocguides/student/) which will allow you to understand all this process and how the program works overall. Refer to its left side menu to quick access sections that may interest you the most, although we recommend you to read everything.  
  
#### 1. Discuss the project idea with the mentor(s)

This is a required step unless you have dived in into the existing codebase and understood everything perfectly (very hard) and the idea you prefer is on the list below.

If your idea is not listed, please discuss it with the mentors in the available [contact channels](https://github.com/metacall/gsoc-2021#find-us). We're always open to new ideas and won't hesitate on choosing them if you demonstrate to be a good candidate!  
  
#### 2. Understand that

- You're committing to a project and we may ask you to publicly publish your weekly progress on it.
- It's the second year Metacall is joining the GSoC program and we will ask you to give feedback on our mentorship and management continuously.
- You wholeheartedly agree with the [code of conduct](https://github.com/metacall/core/blob/develop/.github/CODE_OF_CONDUCT.md).
- You must tell us if there's any proposed idea that you don't think would fit the timeline or could be boring (yes, we're asking for feedback).
  
#### 3. Fill out the application form

We recommend you to follow [Google's guide to Writing a Proposal](https://google.github.io/gsocguides/student/writing-a-proposal) as we won't be too harsh on the format and we won't give any template. But hey, we're giving you a starting point!

You can send the proposal link to any readable format you wish: Google Docs, plain text, in markdown... and preferably hosted online, accessible with a common browser **without downloading anything**.

You can also ask for a review anytime to the community or mentor candidates before the student application deadline. It's much easier if you get feedback early than to wait for the last moment.
  

## Project Ideas

You can also propose your own.

### Cross-Platform Builds

Skills: C/C++, Bash, Batch, Guix, DevOps

Description:
MetaCall has multiple runtimes embedded on it and one of its objectives is to be as cross platform as possible. Each runtime has its own dependencies which create a huge dependency tree sometimes. This is a big complexity that is difficult to handle. Currently we are using Guix as our build system in Linux and we achieved to compile MetaCall and make it completely self-contained (in Linux for amd64 at the moment of writting). This allows MetaCall to be installed even in a BusyBox and work properly without any other system dependency. Current implementation of the build system consist of 3 repositories:

 - Distributable Linux: https://github.com/metacall/distributable-linux
 - Distributable Windows: https://github.com/metacall/distributable-windows
 - Distributable MacOs: https://github.com/metacall/distributable-macos

The objective of this idea is to improve the cross-platform support for the main supported platforms:
 - Implementing the build script for MacOs from scratch with automated CI, in a similar way to how Windows distributable has been done. Add tests and improve the install script for Linux [by adding support to MacOs](https://github.com/metacall/install/blob/master/install.sh#L213).
 - Improving the language support[[1]](https://github.com/metacall/distributable-windows/issues/14)[[2]](https://github.com/metacall/distributable-windows/issues/13) for Windows with their [respective tests](https://github.com/metacall/distributable-windows/blob/master/test.bat) and [improving the install script](https://github.com/metacall/install/blob/282d193329c6c9bbfca86b5ce4c7db8679f67c88/install.ps1#L158) for Windows.
 - Adding [new architectures](https://www.gnu.org/savannah-checkouts/gnu/autoconf/manual/autoconf-2.70/html_node/Specifying-Target-Triplets.html#Specifying-Target-Triplets) to Linux with their respective tests (for this it is possible to use Docker multi-architecture for supporting those tests).

Resources:
 - Guix Build System options: https://guix.gnu.org/manual/en/html_node/Additional-Build-Options.html
 - Docker Multi-Arch support: https://docs.docker.com/desktop/multi-arch/


### Builder

Skills: Go, Docker, BuildKit

TODO

### Deploy CLI

Skills: TypeScript

TODO

### FaaS Reimplementation

Skills: TypeScript

TODO

### MetaCall CLI Bootstrapping / Refactor

Skills: C/C++, Python or NodeJS

TODO

### Runtime Manager

TODO

### Run MetaCall in Browser

TODO

### Polyglot debugger

TODO

### Polyglot IDE

TODO

## Find Us

Telegram:
<a href="https://t.me/joinchat/BMSVbBatp0Vi4s5l4VgUgg" alt="Telegram"><img src="https://img.shields.io/static/v1?label=metacall&message=join&color=blue&logo=telegram&style=flat" /></a>

Discord: 
  <a href="https://discord.gg/upwP4mwJWa" alt="Discord"><img src="https://img.shields.io/discord/781987805974757426?label=discord&style=flat" /></a>

Matrix (bridged with Telegram):
  <a href="https://matrix.to/#/#metacall:matrix.org" alt="Matrix"><img src="https://img.shields.io/matrix/metacall:matrix.org?label=matrix&style=flat" /></a>
