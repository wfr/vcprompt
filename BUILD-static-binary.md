## Compiling a static binary on Debian GNU/Linux

    apt-get install musl-tools
    CC="musl-gcc -static" ./configure && make
    upx -9 vcprompt

This yields a static binary `vcprompt` that should work on most systems.