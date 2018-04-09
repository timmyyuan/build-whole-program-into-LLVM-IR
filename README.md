# ChromiumBuild
Here is some notes for how to build the chromium project into a single LLVM IR bitcode file (ONLY on Linux). Generally, get a integrate LLVM IR bitcode has several purposes such as whole program analysis or optimization.
build LLVM with gold plugins
--
LLVM official organization recommand users to use gold plugins to build a whole project into LLVM bitcodes. There is also some useful opensource tools to achieve this goal, e.g. wllvm (a refinement python script can be found on github). The different between gold plugins and wllvm is the former performs a real linking process with LTO (link time optimization) and the latter simply use llvm-link to connect all intermediate bitcodes in series. No matter which method be chosed, the difference of the bitcode they produce is very small in practice.

check out the chromium project
--

chromium official homepage detailed records how to checkout and download the chromium project. To avoid unnecessary compatibility problems, here we prefer Ubuntu 16.04 (x64) to build the newest chromium.

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
