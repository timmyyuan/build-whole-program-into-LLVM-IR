# ChromiumBuild

Here is some notes for how to build the chromium project into a single LLVM IR bitcode file (ONLY on Linux). Generally, get a integrate LLVM IR bitcode has several purposes such as whole program analysis or optimization.

build LLVM with gold plugins
--

LLVM official organization recommand users to use gold plugins to build a whole project into LLVM bitcodes. There is also some useful opensource tools to achieve this goal, e.g. wllvm (a refinement python script can be found on github). The different between gold plugins and wllvm is the former performs a real linking process with LTO (link time optimization) and the latter simply use llvm-link to connect all intermediate bitcodes in series. No matter which method be chosed, the difference in the bitcodes they produce is very small in practice.

### download and build binutils.
```sh
sudo apt install bison flex libncurses5-dev texinfo
git clone --depth 1 git://sourceware.org/git/binutils-gdb.git binutils
mkdir binutils_build && mkdir binutils_install && cd binutils_build
../binutils/configure --disable-werror --prefix=/path/to/binutils_install
make install
```
### build LLVM with binutils header.
```sh
sudo apt install subversion cmake zlib1g zlib1g-dev
cd /where/you/want/llvm/to/live
svn co http://llvm.org/svn/llvm-project/llvm/trunk llvm
cd llvm/tools
svn co http://llvm.org/svn/llvm-project/cfe/trunk clang
cd /where/you/want/to/build/llvm
mkdir build && cd build
cmake /where/you/want/llvm/to/live -DLLVM_BINUTILS_INCDIR=/path/to/binutils/include
make -j8
```
### add newest binutils and newest LLVM to envirnoment variables.
```sh
vim ~/.bashrc
export PATH=/path/to/binutils_install/bin:$PATH # add this line to bashrc
export PATH=/path/to/llvm/build/bin:$PATH       # add this line to bashrc
source ~/.bashrc
```
### test (optional)

we use sed as a benchmark to test whether the gold plugins work correctly in LLVM.
```sh
# check clang and binutils version
clang --version
binutils --version
# if the work flow above is correct
cd /path/to/workspace
mkdir sed_build && cd sed_build
/path/to/sed/source/configure CC=clang LDFLAGS='-flto -fuse-ld=gold -Wl,-plugin-opt=save-temps'
make
```
If everything is ok, sed.0.0.preopt.bc can be found under the build directory. (sepecifically in /path/to/sed_build/sed)

check out the chromium project
--

chromium official homepage detailed records how to checkout and build the chromium project. To avoid unnecessary compatibility problems, here we prefer Ubuntu 16.04 (x64) to build the newest chromium.

generate build system files into the build directory
--

something here ~

modify build system files
--

something here ~

building
--

something here ~

some issues
--

something here ~
