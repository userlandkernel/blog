# Setting up the perfect development environment on iOS 12

Many of the iOS users do not know that it is possible to develop iOS apps on-device while jailbroken.  
Not only theos, but also general development environments can be built.  
One would need a combination of packages of which some are not available in the famous AppStores (Cydia / Sileo) for jailbroken users.  
Therefore I thought it would help a lot of developers to setup a great development environment on iOS

## Installing the compilers and scripting languages

1. LLVM and Clang.  
**REPO: *http://cydia.jskjdskjds.com***.  
This amazing toolchain lets you built lowlevel binaries in C, C++ or assembly.  
It is the perfect toolchain for the system engineer, swift might be supported but I have not checked that yet.  
LLVM and clang projects depend on an SDK to be installed, this article will explain how to install an SDK later.  

2. Python 2 and python 3.  
**REPO: *http://cydia.jskjdskjds.com***.   
Believe it or not, python is the best scripting language for automating operations on your device.  
The possibilities of python are endless and the language is easy to understand.  
One disadvantage of python is the format in which you write it needs to be clean, otherwise python won't compile your script.  

3. Lua.  
**REPO: *http://cydia.jskjdskjds.com***.  
Used by universities and gaming engines, lua is a great scripting language that makes lowlevel programming easy.  
Security however is not its focus, that's left to the programmer to think about.  

4. Node.  
**REPO: *http://cydia.jskjdskjds.com***.   
Node is the famous v8 based javascript interpreter / compiler.  
It is an amazing asset in writing easy and manageable projects in short time.  
Node for example can be used for implementing websocket servers.  

5. Ruby.  
**REPO: *http://cydia.jskjdskjds.com***.  
The popular hacker language runs on iOS too. Enough said I guess.  

6. Perl.  
**REPO: *http://cydia.jskjdskjds.com***.  
Cryptographers seem to love perl, cpanimus the package manager for perl runs fine on iOS too.  
Apple also has some love relation with perl, even on production devices there are some leftover scripts written in perl for automation.  



## Installing debuggers on iOS 12

1. The Low Level Debugger.  
**REPO: *http://cydia.jskjdskjds.com***.  
LLDB is a debugger optimized for use with LLVM and Clang, but it runs fine with GNU programs too ofcourse.  
LLDB is amazing for debugging tweaks and processes on iOS.  
The debugger implements on task_for_pid and lets you attach to processes, pause execution, inspect registers and memory and disassemble.  
The debugger is a great tool for those who love making tweaks, but doesn't support kernel debugging.  

2. Cycript.  
**REPO: *http://cydia.jskjdskjds.com***.    
Cycript is a debugger that can be injected into processes.  
It let's you remotely debug them and inspect the runtime.  
It is the easiest debugger one can get, as it is written to support javascript-like syntax because it is based off JavaScriptCore.  
Tweak developers somewhat are married to Cycript, it is definitly worth a try for people that are new to debugging.

3. Kernel utilities.  
Kern-utils is not a debugger but lets you inspect, dump and patch kernel memory. 
It is a great tool for those who want to play around with kernel patches to make the jailbreak have even more freedom and features.  
Patching or reading invalid memory will obviously result in a panic.  

## Installing an SDK and documentation

## Installing build tools

1. GNU Make.  
Make allows you to create files for building your projects.  
You can divide your project in different products and also manage installation and removal of the project with make.  
It saves you a lot of time, as you do not have to memorize the build commands and flags anymore.  
Most software sources use make files these days.


2. Autoconf.  
With autoconf one can generate makefiles accross different platforms as you can not expect a marriage between windows and the komodore64.  
But with autoconf, that will not be an issue ;)


3. Automake.  
TBH its probably useful, but I do not know what this does exactly.


4. Libtool.  
Utility used in building software from source for linking and editing libraries, I guess?


5. GIT.  
The portal to the opensource paradise (just kidding).  
It's the best versioning tool at the moment and let's you sync software with repositories.  
Each commit to a project is saved and you can organize your development stages in different branches.  
GitHub is perhaps the most popular platform for creating and downloading software repositories, but GitLab is a great alternative.  



6. Subversion.  
Well, some old code still is on the svn repositories that were popular before git became mainstream.  
You might need it, or not.  


## Installing management tools

1. Htop.  
Htop is a taskmanager commandline ui.  
Whenever you need to kill or signal a process htop will be of great help.  

2. LocalSSH.  
I really thank the developer for making this tweet, because NewTerm 2 has many bugs making it hard to use for someone who spends much time in terminals.  
Maybe the bugs are patched now, go check it out, but I prefer connecting locally through termius from the appstore.  

3. 
