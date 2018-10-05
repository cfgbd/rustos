# RustOS实验报告

## 成员

- 第二小组
    - 贾越凯 2015011335
    - 寇明阳 2015011318
    - 孔彦 2015011349

## 实验内容

### 问题

#### 1：由于项目目录含有空格，导致编译出错

* 错误信息：

    ```
    error: failed to run `rustc` to learn about target-specific information

    Caused by:
    process didn't exit successfully: `rustc - --crate-name ___ --print=file-names --sysroot /home/lenovo/Desktop/OS training/RustOS/kernel/target/sysroot --target '/home/lenovo/Desktop/OS training/RustOS/kernel/x86_64-blog_os.json' --crate-type bin --crate-type rlib --crate-type dylib --crate-type cdylib --crate-type staticlib --crate-type proc-macro` (exit code: 1)
    --- stderr
    error: multiple input filenames provided
    ```

* 解决办法：

    将项目路径从`/home/lenovo/Desktop/OS training/RustOS/`改为`/home/lenovo/Desktop/os_training/RustOS/`

#### 2：在变更路径之后，由于rustup环境默认选择的linux运行系统为stable，与项目配置冲突，导致编译出错

* 错误信息：

    ```
    error: unable to get packages from source

    Caused by:
    failed to parse manifest at `/home/lenovo/.cargo/registry/src/github.com-1ecc6299db9ec823/bootloader-0.3.1/Cargo.toml`

    Caused by:
    the cargo feature `publish-lockfile` requires a nightly version of Cargo, but this is the `stable` channel
    Makefile:129: recipe for target 'kernel' failed
    make: *** [kernel] Error 1
    ```

* 解决方法：

    运行指令`rustup override set nightly`或`rustup override set nightly-2018-09-18`

* 运行效果：

    ```
    info: using existing install for 'nightly-2018-09-18-x86_64-unknown-linux-gnu'
    info: override toolchain for '/home/lenovo/Desktop/os_training/RustOS/kernel' set to 'nightly-2018-09-18-x86_64-unknown-linux-gnu'

    nightly-2018-09-18-x86_64-unknown-linux-gnu unchanged - rustc 1.30.0-nightly (2224a42c3 2018-09-17)
    ```
    *解释：可以看出rustup对指定的路径设置了要使用nightly版本的工具链，而不是对命令行的环境*

#### 3：macos 下 arch=x86_64 编译 bootloader 时出现 \`ld.bfd\` not found 的错误

* 错误信息：

    ```
    error: linker `ld.bfd` not found
    |
    = note: No such file or directory (os error 2)

    error: aborting due to previous error

    error: Could not compile `bootloader`.

    To learn more, run the command again with --verbose.
    make: *** [kernel] Error 1
    ```

* 解决方法：

    1. 安装 `x86_64-unknown-linux-gnu` 工具链：

        ```
        brew tap SergioBenitez/osxct
        brew install x86_64-unknown-linux-gnu
        ```

    2. 前往 `cargo` 安装了 `bootloader` 的目录(形如 `$HOME/.cargo/registry/src/github.com-1ecc6299db9ec823/bootloader-0.3.1`)；

    3. 修改该目录下的 `x86_64-bootloader.json` 文件，将

        ```
        "linker" : "ld.bfd"
        ```

        改为

        ```
        "linker" : "x86_64-unknown-linux-gnu-ld.bfd"
        ```