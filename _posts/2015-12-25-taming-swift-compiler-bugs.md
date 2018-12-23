---
id: 733
title: Taming Swift compiler bugs
date: 2015-12-25T21:58:51+00:00
author: tomasz
layout: post
guid: http://codica.pl/?p=733
permalink: /2015/12/25/taming-swift-compiler-bugs/
excerpt_separator: <!--more-->
categories:
  - Blog Entry
tags:
  - Swift
  - Swift 2
  - swiftc
  - Xcode
  - Xcode 7
---
Xcode 7.2 has a bug which results in compilation failure of project having +1500 Swift files. It seems that _swiftc_ is buggy, but you can try to bypass that behaviour by using my wrapper [script](https://github.com/tgebarowski/swiftc-wrapper). Read the whole post if you are interested in what happens under the hood.

<!--more-->

## Background

I will start with a short story. I'm working on a complex project which has +1500 Swift files and mixes both Objective-C and Swift code. The project is written in Swift 1.2 and still uses Xcode 6.4. Migrating such a big project to Swift 2 is not a trivial task, automatic conversion provided by Xcode 7.2 (_swift-update_ tool) takes several hours and it still can solve only basic issues. The rest has to be solved manually, so it is not only a time consuming task, but also brings higher regression risk. Surprisingly, even after fixing all migration related issues, the app still didn't compile. It looked like the compiler didn't generate a Swift umbrella header that was included by legacy Objective-C code to allow using Swift classes. Build procedure failed on copying header from project _Intermediates_ directory, because it was simply not generated:

`ditto: can't get real path for source '/Users/MyUsername/Library/Developer/Xcode/DerivedData/MyAppName-fwxrneawhopjkqcpeoykduytrgwv/Build/Intermediates/MyAppName.build/Debug-iphoneos/MyAppName.build/Objects-normal/arm64/MyAppName-Swift.h'
Command /usr/bin/ditto failed with exit code 1`

In the beginning this issue sounded familiar and there were already several similar problems on [StackOverFlow](http://stackoverflow.com/questions/24062618/swift-to-objective-c-header-not-created-in-xcode-6). Most of them suggested error in _some_ Swift file which was not reported by Xcode and didn&#8217;t result in successful compilation and hence umbrella header was missing. After spending several hours on finding the problem with some _&#8220;corrupted&#8221;_ Swift file, testing various compilation flags, deleting DerivedData and such, I was still not able to move forward.

Quite accidentally I&#8217;ve noticed that there was a difference when invoking project compilation with and without **whole-module-optimization** parameter.
  
The former resulted in almost instant compilation of Swift files, while the latter a very long compilation of one Swift file after another, resulting in generating object files for each Swift file, but still without generating any Objective-C umbrella header. I decided to check the first issue as I had a gut feeling that this could be a compiler bug.

> It is worth to know that when turned on **-whole-module-optimization** swiftc is not compiling each Swift file after another, but takes all Swift files as an input and generates one merged binary object file. This allows the swiftc to perform better optimization based on all Swift files passed as an input, rather than individual file.

To confirm that this is a _swiftc_ bug I decided to isolate the problem. I wrote a simple Python script that generated N pure Swift classes having one method and tried to compile them using command line version of _swiftc_. When reaching about 2000 Swift files I was able to reproduce the issue with **-whole-module-optimization** option. It definitely looked like a bug, as when swiftc got less than _X_ Swift files to compile, it took a couple of minutes to compile them all and generate both umbrella header and object file. When reaching certain limit of files, _swiftc_ was simply returning immediately, unfortunately without any error code.

## Finding the solution 

Inspired by recent open sourcing I decided to dive into _swiftc_ source code. First I&#8217;ve downloaded latest version of _swiftc_ from github repo and compiled it manually to check out if bug was not fixed recently. Unfortunately it didn&#8217;t help, the problem was still there.

After some debugging I found out that the culprit was **posix_spawn** invoked from _Task::execute()_ (_lib/Basic/Unix/TaskQueue.inc:142_), it returned error code 7, which suggested _[EFAULT]_ aka _Path, argv, or envp point to an illegal address_. I checked both _argv_ and _envp_ and they contained correct entries, didn&#8217;t look like corrupted. I think that reason for _posix_spawn()_ failure could be too long list of arguments stored in _argvp_ (about 5900 entries, which was much bigger than original list of input arguments passed to _swiftc_ &#8211; about 2000 args). The situation could be solved by blocking _posix_spawn_ (_#undef HAVE\_POSIX\_SPAWN_) and using _fork_ function, but this would require _swiftc_ recompilation and was not acceptable.

All of the above gave me some hint, which was that the problem was with parallel execution of compiler jobs and spawning the processes. I decided to check what happened if forcing _swiftc_ to use one worker thread (_-num-threads 1_). This didn&#8217;t help, but what surprisingly worked and allowed skipping process spawning logic was setting _-num-threads 0_ in command line argument.

Knowing how to workaround the issue I decided to persuade Xcode to pass that argument to _swiftc_. It seems that by default Xcode inserts _-num-threads_ equal to number of processor cores in the machine.

Setting:

`defaults write com.apple.dt.Xcode IDEBuildOperationMaxNumberOfConcurrentCompileTasks 0`

didn&#8217;t work, because default value was still passed to _swiftc_. Invoking _xcodebuild_ with _IDEBuildOperationMaxNumberOfConcurrentCompileTasks_ parameter worked for other values, but was not possible with 0. Adding _-num-threads 0_ to _SWIFT COMPILER FLAGS_ in Xcode did not work, because Xcode inserted its version of _-num-threads_ just after value from _SWIFT COMPILER FLAGS_ and it got overwritten. I knew that I was very close, but I still could not get it working.

## Bypassing the compiler 

Finally I came up with a \*brilliant\* idea of wrapping _swiftc_ with a Python script which would simply act as a proxy and append -num-threads 0 to all what was passed to it as input arguments.

I&#8217;ve noticed that Xcode uses _swiftc_ from:

`
/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xc/toolchain/usr/bin
` 

which was a symbolic link to _swift_ binary file in the same directory. I removed _swiftc_ symlink from there and wrote a simple wrapper that was invoking swift. Unfortunately this didn&#8217;t work. I checked _swift_ source code and found out that swift detects how it is executed. If it is executed through a symbolic link its _argv[0]_ is named _swiftc_ and command line version logic is invoked; when invoked directly from swift binary, Swift interpreter is started. The latter is what happened when I invoked swift from the wrapper script. I managed to solve this issue by creating a symbolic link in /usr/local/bin which pointed to _swift_ in:

_/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/swift_

and invoked the compiler through my symbolic link.

My idea worked pretty well, all Swift files were compiled successfully, umbrella header with Swift classes for Objective-C code was generated and everything looked like I was going to see my app running in the simulator. Unfortunately, there was a linker that reported a problem, and this is when I discovered a second bug in Xcode, which can be reproduced easily when compiling the project with _-whole-module-optimization_.

The problem was that the linker expected object file to be generated for each Swift file. Because the project was compiled with _-whole-module-optimization_ only one big object file with all symbols from Swift classes was generated. I checked how Xcode invokes the linker and I&#8217;ve noticed that it passes to it a file suffixed with _*.LinkFileList_ which is stored inside _DerivedData/../Intermediates/../Objects-normal/[ARCH]/_. The file indeed contained a list of all object files, so it looked like it was wrongly generated.

I came up with another idea. Why not to reuse my Python wrapper and modify _*.LinkFileList_ file? I replaced all missing object files from that file, with a path to a file being output from _swiftc_ invoked with _-whole-module-optimization_. But how this could be automated? I could not truncate _*.LinkFileList_ file and add merged object file path because it contained also objects from Objective-C compiler. Finally I came up with an idea of using input parameters from _swiftc_ command and retrieve the names of Swift files from that argument list. After some parsing and adapting file names it worked pretty well. I was able to successfully compile Swift 2 version of my project.

I reported this problem to Apple and it seems that it affects more people. You can follow this issue [here (SR-280)](https://bugs.swift.org/browse/SR-280)

Python script acting as a wrapper for _swiftc_ is published on my [github account](https://github.com/tgebarowski/swiftc-wrapper)

You may also want to ask why not dividing the project into frameworks and avoid all that problems? Unfortunately this is not possible as I have to still target iOS 7 devices which doesn&#8217;t support dynamic Swift frameworks.

If you like this post, please follow me on twitter [@tgebarowski](https://twitter.com/tgebarowski)