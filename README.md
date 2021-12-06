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
- It's the first year Metacall is joining the GSoC program and we will ask you to give feedback on our mentorship and management continuously.
- You wholeheartedly agree with the [code of conduct](https://github.com/metacall/core/blob/develop/.github/CODE_OF_CONDUCT.md).
- You must tell us if there's any proposed idea that you don't think would fit the timeline or could be boring (yes, we're asking for feedback).
  
#### 3. Fill out the application form

We recommend you to follow [Google's guide to Writing a Proposal](https://google.github.io/gsocguides/student/writing-a-proposal) as we won't be too harsh on the format and we won't give any template. But hey, we're giving you a starting point!

You can send the proposal link to any readable format you wish: Google Docs, plain text, in markdown... and preferably hosted online, accessible with a common browser **without downloading anything locally**.

You can also ask for a review anytime to the community or mentor candidates before the student application deadline. It's much easier if you get feedback early than to wait for the last moment.
  

## Project Ideas


### Cross-Platform Builds

Skills: C/C++, Guix, DevOps

Description:
MetaCall has multiple runtimes embedded on it and one of its objectives is to be as cross platform as possible. Each runtime has its own dependencies which create a huge dependency tree sometimes. This is a big complexity that is difficult to handle. Our first approach was to Dockerize all dependencies, building them manually, taking care of `LDFLAGS` in order to make the distributable portable at the same time. After this, we tried Guix to implement our build system and we achieved to compile MetaCall completely and make it completely self-contained (in Linux for amd64). This allows MetaCall to be installed even in a BusyBox and work properly without any other system dependency. We have started to support cross-platform builds with Guix but the development has stalled. The objective of this task is to achieve cross-platform builds for Linux, Windows and MacOs, and multiple hardware architectures. Guix supports cross-compiling but it is sometimes restricted due to its nature. One option we propose is to override the `gnu-build-system` of Guix by using Zig compiler and allow Cross-Compiling by means of its frontend. But, there are many other options to do it and we are open to them. Maybe if it can be achieved with a different package manager we are open to it too.

Resources:
 - Initial Docker Cross-Platform Build Approach: https://github.com/metacall/distributable/tree/feature/build-scratch/linux
 - First steps in Cross-Platform Build for Guix: https://github.com/metacall/distributable/tree/feature/build-guix-cross
 - Base implementation of Cross-Platform support in `metacall/distributable`: https://github.com/metacall/distributable/blob/f14cbdcfe3bcb85e0331172e0dbb512e2f21350a/Makefile#L31 and https://github.com/metacall/distributable/blob/f14cbdcfe3bcb85e0331172e0dbb512e2f21350a/scripts/build.sh#L23
 - Guix Build Options Documentation: https://guix.gnu.org/manual/en/html_node/Additional-Build-Options.html
 - Cross-compiling with Zig: https://andrewkelley.me/post/zig-cc-powerful-drop-in-replacement-gcc-clang.html


## Find Us

Telegram:
<a href="https://t.me/joinchat/BMSVbBatp0Vi4s5l4VgUgg" alt="Telegram"><img src="https://img.shields.io/static/v1?label=metacall&message=join&color=blue&logo=telegram&style=flat" /></a>

Discord: 
  <a href="https://discord.gg/upwP4mwJWa" alt="Discord"><img src="https://img.shields.io/discord/781987805974757426?label=discord&style=flat" /></a>

Matrix (bridged with Telegram):
  <a href="https://matrix.to/#/#metacall:matrix.org" alt="Matrix"><img src="https://img.shields.io/matrix/metacall:matrix.org?label=matrix&style=flat" /></a>
