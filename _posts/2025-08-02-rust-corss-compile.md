# rust跨平台编译

## rust 的 target

先安装需要的target

```cmd
rustup target add armv7-unknown-linux-gnueabihf
```

指定对应的target的config

```toml
[target.armv7-unknown-linux-gnueabihf]
linker="/home/mincong/Work/Buildroot_2020.02.x/output/host/bin/arm-buildroot-linux-gnueabihf-gcc"
```

然后

```cmd
cargo build --target armv7-unknown-linux-gnueabihf
```

## lib depending  

然而他好像不会找cc的sysroot，实际上，还是要拿一次sysroot

指定各种PKG_CONFIG、OPENSSL（actix-web\reqwest\openssl-sys) 需要

理论上，openssl应该让pkgconfig搞定，但哪怕我指定了staging的pkgconfig，一样没用，我已经看了openssl.pc，路径应该是没错的，但还是要手动指定OPENSSL_DIR

``` bash
#export SYSROOT=/home/mincong/Buildroot_2020.02.x/output/host/arm-buildroot-linux-gnueabihf/sysroot/
export LC_ALL=zh_CN.utf-8

export SYSROOT=$(/home/mincong/Work/Buildroot_2020.02.x/output/host/bin/arm-buildroot-linux-gnueabihf-gcc -print-sysroot)
export TARGET_CC="/home/mincong/Work/Buildroot_2020.02.x/output/host/bin/arm-buildroot-linux-gnueabihf-gcc --sysroot=$sysroot"
export CFLAGS="--sysroot=$sysroot -I/home/mincong/Work/Buildroot_2020.02.x/output/staging/usr/include/"
export OPENSSL_DIR=/home/mincong/Work/Buildroot_2020.02.x/output/staging/usr
export OPENSSL_LIB_DIR=/home/mincong/Work/Buildroot_2020.02.x/output/staging/usr/lib
export PKG_CONFIG_SYSROOT_DIR=/home/mincong/Work/Buildroot_2020.02.x/output/staging/usr/lib/pkgconfig

cargo build --target=armv7-unknown-linux-gnueabihf -p sdf-test
```

## bindgen

跨平台，会提示一个bits/libc-header-start.h找不到，改了很多环境变量都不行，又要强制指定一下clang_arg

``` rust
let bindings = bindgen::Builder::default()
        // The input header we would like to generate
        // bindings for.
        .clang_arg("-I/home/mincong/Work/Buildroot_2020.02.x/output/staging/usr/include")
        .header("./include/GMT0033.h")
        // Finish the builder and generate the bindings.
        .generate()
        // Unwrap the Result and panic on failure.
        .expect("Unable to generate bindings");
···

特别地，在Windows，需要借助msys解决clang的问题

msys2上要安装libclang的库，具体那个忘了

在环境变量设置：

``` bash
LIBCLANG_PATH=D:\msys64\mingw64\bin;PATH=D:\msys64\mingw64\bin
```

## build.rs

其实很多应该可以用build.rs里面的println!("cargo:") 来做环境设置

```rust
println!("cargo:include=/home/mincong/Work/Buildroot_2020.02.x/output/staging/usr/include");
println!("cargo:rerun-if-changed=./include/GMT0033.h");
```

* cargo:rustc-link-search=<path>  添加库搜索路径  -L /path/to/libs

* cargo:rustc-link-lib=<name> 链接指定的库（动态或静态）  -l<name>

* cargo:rustc-cfg=<feature>   启用条件编译配置    --cfg <feature>

* cargo:rustc-env=VAR=value   设置编译时环境变量  -

* cargo:rerun-if-changed=<file>   监控文件变化，触发 rebuild

诸如此类
