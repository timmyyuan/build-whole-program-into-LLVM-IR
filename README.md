# ChromiumBuild

Here is some notes for how to build the chromium project into a single LLVM IR bitcode file (ONLY on Linux). Generally, get a integrate LLVM IR bitcode has several purposes such as whole program analysis or optimization. Chromium official homepage detailed records how to checkout and build the chromium project. To avoid unnecessary compatibility problems, here we prefer Ubuntu 16.04 (64 bit) to build the newest chromium.

build LLVM with gold plugins
--

LLVM official organization recommand users to use gold plugins to build a whole project into LLVM bitcodes. There is also some useful opensource tools to achieve this goal, e.g. wllvm (a refinement python script can be found on github). The different between gold plugins and wllvm is the former performs a real linking process with LTO (link time optimization) and the latter simply use llvm-link to connect all intermediate bitcodes in series. No matter which method be chosed, the difference in the bitcodes they produce is very small in practice.

### download and build binutils.
```sh
# some necessary pre-requisite
sudo apt install bison flex libncurses5-dev texinfo
# get the trunk version of binutils
git clone --depth 1 git://sourceware.org/git/binutils-gdb.git binutils
# create a build directory
mkdir binutils_build && mkdir binutils_install && cd binutils_build
# configuration
../binutils/configure --disable-werror --prefix=/path/to/binutils_install
# build/compile
make install
```
### build LLVM with binutils header.
```sh
# some necessary pre-requisite
sudo apt install subversion cmake zlib1g zlib1g-dev
# get the trunk version of LLVM and clang
cd /where/you/want/llvm/to/live
svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm
cd llvm/tools
svn co http://llvm.org/svn/llvm-project/cfe/trunk clang
cd /where/you/want/to/build/llvm
# create build and install directories
mkdir llvm_build && mkdir llvm_install && cd llvm_build
# configuration with binutils header
cmake /where/you/want/llvm/to/live -DLLVM_BINUTILS_INCDIR=/path/to/binutils/include -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_INSTALL_PREFIX=/path/to/llvm_install ../llvm
# build/compile
make -j8
```
the following cmake flags is opional :
```sh
-DLLVM_ENABLE_ASSERTIONS=On
-DLLVM_ENABLE_RTTI=On
```

### add newest binutils and newest LLVM to envirnoment variables.
```sh
vim ~/.bashrc
# use (ESC + a) to insert sentences to current file in vi/vim
export PATH=/path/to/binutils_install/bin:$PATH # add this line to bashrc
export PATH=/path/to/llvm/build/bin:$PATH       # add this line to bashrc
# use (ESC + :wq) to quit vi/vim
```
then type the command to shell
```sh
source ~/.bashrc
```
to evaluate the envirnoment variables, reopen shell is also workful.

### test (optional)

we use sed as a benchmark to test whether the gold plugins work correctly in LLVM.
```sh
# check the envirnoment variables are correct.
clang --version
binutils --version
# switch to workspace where to build sed.
cd /path/to/workspace
# create a build directory.
mkdir sed_build && cd sed_build
# configuration
/path/to/sed/source/configure CC=clang LDFLAGS='-flto -fuse-ld=gold -Wl,-plugin-opt=save-temps'
# build/compile
make
```
If everything is ok, sed.0.0.preopt.bc can be found under the build directory. (sepecifically in /path/to/sed_build/sed)

check out the chromium project
--

The first step of check out is clone depot_tools and configure depot_tools according to offical website of chromium. (https://chromium.googlesource.com/chromium/src/+/master/docs/linux_build_instructions.md)
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
