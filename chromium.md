# Build chromium into LLVM IR

Here is some notes for how to build the chromium project into a single LLVM IR bitcode file (ONLY on Linux). Chromium official homepage detailed records how to checkout and build the chromium project. To avoid unnecessary compatibility problems, here we prefer Ubuntu 16.04 (64 bit) to build the newest chromium.

some useful reference:
[offical : build chromium on linux](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_build_instructions.md)
[offical : build chromium by clang](https://chromium.googlesource.com/chromium/src/+/master/docs/clang.md)
[build chromium into LLVM IR](https://github.com/SVF-tools/SVF/wiki/Compiling-Chrome-using-flto)

check out the chromium project
--

The first step of check out is clone depot_tools and configure depot_tools according to offical website of chromium. 
```
cd where-to-live-depot_tools
git clone https://chromium.googlesource.com/chromium/tools/depot_tools.git
export PATH="$PATH:/path/to/depot_tools"
```
#### 1. from googlesource

```sh
mkdir ~/chromium && cd ~/chromium
fetch --nohooks chromium
cd src && ./build/install-build-deps.sh
gclient runhooks
```

#### 2. from github

For chinese mainland users, we perfer to clone code from a mirror site instead of googlesource. the mirror of chromium can be found easily on github
```sh
git clone https://github.com/chromium/chromium.git src
gclient config --spec 'solutions = [
  {
    "url" : "https://github.com/chromium/chromium.git",
    "managed" : False,
    "name" : "src",
    "custom_deps" : {},
  },
]'
gclient sync --nohooks
cd src && ./build/install-build-deps.sh
gclient runhooks
```
Once ./build/install-build-deps.sh be executed, the build envirnoment of chromium is complete. The operating system is running in low graphics mode when we reboot Ubuntu. Switch to terminal (Ctrl + F1) and type
```sh
sudo apt update && sudo apt upgrade
```
to fix it.

generate build system files into the build directory
--
assume we are undering the src directory (the root of chromium project).
```sh
gn args out/mybuild
```
this command will bring users to a vi/vim editor, type the following configuration for compiling:
```sh
# use (ESC + a) to insert sentences to current file in vi/vim
clang_base_path = "/path/to/llvm_build"
binutils_path = "/path/to/binutils_install/bin"
clang_use_chrome_plugins = false
is_component_build = true
enable_nacl = false
is_debug = false
symbol_level = 0
use_lld = false
# use (ESC + :wq) to quit vi/vim
```
disable NACL (native client) is necessary because NACL will use its built-in compiler which do not support '-flto'. 

modify build system files
--

c/cxx/link flags should be changed because we will save temporary bitcodes in compile time. 
Use your favorite editor to open directory out/mybuild then
```sh
replace all appears of "${ldflags}" to "${ldflags} -flto -fuse-ld=gold -Wl,plugin-opt=save-temps" 
replace all appears of "${cflags_c}" to "${cflags_c} -flto"
replace all appears of "${cflags_cxx}" to "${clags_cxx} -flto"
```
If you want to add some flags yourself, you can change the build command as below, for example
```sh
replace all appears of "${cflags_c}" to "${cflags_c} -flto -g3 -O0"
```

building
--

```sh
ninja -C out/mybuild chrome -j 16
```
use -v to obvious the building flows.

some issues
--

something here ~
