# Build whole programs into LLVM IR

Here is some notes for how to build the projects into a single LLVM IR bitcode file. Generally, get a integrate LLVM IR bitcode has several purposes such as whole program analysis or optimization. Based on information on the network, there are two ways, named gold plugin and whole-program-llvm respectively, to achieve this goal.

LLVM official organization introduced the gold plugin to support LTO (link time optimization) and further be used to build the whole project into LLVM bitcodes. Some useful opensource tools has been published on github, e.g. wllvm (whole-program-llvm, write in python and support Linux and Mac). 

The mainly differences between the gold plugin and wllvm are :
* the gold plugin only support ELF files while wllvm can also support MACHO files.
* wllvm unsupports share libraries and does not need the targets are position independent (i.e. '-fPIC').
* the gold plugin performs a real link process (with LTO, i.e. '-flto') while wllvm simply uses llvm-link (with less optimizations) to connect all intermediate bitcodes produced in compile time. 

More information can be found in reference but the bitcodes (of executables) produced by above methods are extremly similar in practice.

## The gold plugin

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
cmake ../llvm -DLLVM_BINUTILS_INCDIR=/path/to/binutils/include -DCMAKE_BUILD_TYPE=Release -DLLVM_TARGETS_TO_BUILD=X86 -DCMAKE_INSTALL_PREFIX=/path/to/llvm_install
# build/compile
make -j4
```
the following cmake flags are optional :
```sh
-DLLVM_ENABLE_ASSERTIONS=On
-DLLVM_ENABLE_RTTI=On (LLVM Release will disable RTTI by default)
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
/path/to/sed/source/configure CC=clang CFLAGS='-flto' LDFLAGS='-flto -fuse-ld=gold -Wl,-plugin-opt=save-temps'
# build/compile
make
```
If everything is ok, sed.0.0.preopt.bc can be found under the build directory. (sepecifically in /path/to/sed_build/sed)

## whole-program-llvm

### local python environment (optional)
we create isolated python environments for wllvm. Make sure virtualenv already be installed:
```sh
sudo pip install virtualenv 
```
Now we create and activate a new isolated python envirnoment:
```sh
cd /where/we/want/to/live/python/envirnoments
virtualenv pyenv --no-site-packages
source pyenv/bin/activate
```
### install wllvm
fetch wllvm and setup:
```sh
git clone https://github.com/travitch/whole-program-llvm wllvm
cd wllvm && python setup.py install
```
note wllvm is also available on pip.
```sh
pip install wllvm
```
### test (optinal)
build sed and generate the bitcodes like:
```sh
mkdir sed_build && cd sed_build
/path/to/sed/source/configure CC=wllvm
make -j4
cd sed && extract-bc sed
```
Then you can find sed.bc in the same folder as the sed executable.

## Reference

[Getting Started with the LLVM System](http://llvm.org/docs/GettingStarted.html)
<br>[The LLVM gold plugin](https://llvm.org/docs/GoldPlugin.html)
<br>[Compiling Autotooled projects to LLVM Bitcode](http://gbalats.github.io/2015/12/10/compiling-autotooled-projects-to-LLVM-bitcode.html)
<br>[Install LLVM Gold plugin on Ubuntu](https://github.com/SVF-tools/SVF/wiki/Install-LLVM-Gold-Plugin-on-Ubuntu)
<br>[whole-program-llvm](https://github.com/travitch/whole-program-llvm)
