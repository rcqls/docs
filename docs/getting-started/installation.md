# Installation

## Installing V from source (preferred way)

To install from source, you need `git`, `make` and C compiler installed on your system.
V ships with TCC compiler, so most probably you do not need to install C compiler manually, but
if there is an error during the build V, see [C compiler](#c-compiler) section for more details.

### Linux, macOS, *BSD, Solaris, WSL, etc

Run the following in a terminal/shell:

```bash
git clone https://github.com/vlang/v
cd v
make
```

This will create a `./v` executable in the current directory.

> **Tip**
> If the execution ended with an error, then most likely the C compiler is not installed,
> see the [C compiler](#c-compiler) section for more details.

Let us check if it works:

```bash
./v run examples/hello_world.v
```

This command should print `Hello, World!` to the console.

If it doesn't, please see
[Installation Issues](https://github.com/vlang/v/discussions/categories/installation-issues)
for help.

### Windows

Run the following commands in cmd or PowerShell:

```bash
git clone https://github.com/vlang/v
cd v
make.bat # or ./make.bat in PowerShell
```

This will create a `v.exe` executable in the current directory.

> **Tip**
> If the execution ended with an error, then most likely the C compiler is not installed,
> see the [C compiler](#c-compiler) section for more details.

Let us check if it works:

```bash
v run examples/hello_world.v
```

This command should print `Hello, World!` to the console.

If it doesn't, please see
[Installation Issues](https://github.com/vlang/v/discussions/categories/installation-issues)
for help.

### Void Linux

```bash
xbps-install -Su base-devel
xbps-install libatomic-devel
git clone https://github.com/vlang/v
cd v
make
```

### Docker

```bash
git clone https://github.com/vlang/v
cd v
docker build -t vlang .
docker run --rm -it vlang:latest
```

### Docker with Alpine/musl

```bash
git clone https://github.com/vlang/v
cd v
docker build -t vlang --file=Dockerfile.alpine .
docker run --rm -it vlang:latest
```

### Termux/Android

On Termux, V needs some packages preinstalled – a working C compiler, `libexecinfo`,
`libgc` and `libgc-static`.

```bash
pkg install clang libexecinfo libgc libgc-static make git
git clone https://github.com/vlang/v
cd v
make
```

## Installing pre-built binaries

You can download pre-built binaries directly from the <https://vlang.io> website for Linux, macOS, and
Windows or from the [release page](https://github.com/vlang/v/releases).

> **Note**
> The pre-built binaries are not updated as often as the source code, so they may be out of date.
> See [Updating V](#updating-v) section for instructions on how to update V.

## C compiler

V uses C compiler to compile V programs and itself.

The [Tiny C Compiler (TCC)](https://repo.or.cz/w/tinycc.git) is downloaded for you by `make` if
there is a compatible version for your system, and installed under the V `thirdparty` directory.
This compiler is very fast, but does almost no optimizations. It is best for development builds.

If V cannot find a compatible version of tcc, it will try to use the system C compiler.
If you do not have a C compiler installed, follow these instructions:

- [How to install C compiler on Linux and macOS](../meta/how-to-install-c-compiler-on-unix.md)
- [How to install C compiler on Windows](../meta/how-to-install-c-compiler-on-windows.md)

## Symlinking

To have convenient access to the compiler by its name from anywhere, the compiler provides
the handy `symlink` command, which creates a symbolic link `/usr/local/bin/v` to the executable V on
Unix.
On Windows, it adds the path to the V executable to the PATH environment variable.

### On Unix

```bash
sudo ./v symlink
```

### On Windows

On Windows, start a new shell with administrative privileges, for example, by pressing the
<kbd>Windows Key</kbd>, then type `cmd.exe`, right-click on its menu entry, and choose `Run as
administrator`.
In the new administrative shell, cd to the path where you have compiled V, then type:

```bat
v symlink
# or ./v symlink in PowerShell
```

That will make V available everywhere by adding it to your PATH.
Please restart your shell/editor after that, so that it can pick up the new PATH variable.

> **Note**
> There is no need to run `v symlink` more than once – V will still be available, even after
> `v up`, restarts, and so on. You only need to run it again if you decide to move the V repo
> folder somewhere else.

## Updating V

V is constantly evolving, so to use the latest up-to-date version, you need to update V.

To update V, simply run:

```bash
v up
```

## V net.http, net.websocket, `v install`

`net.http`, `net.websocket` module, and the `v install` command may all use SSL.

V comes with a version of `mbedtls`, which should work on all systems. If you find a need to
use OpenSSL instead, you will need to make sure that it is installed on your system, then
use the `-d use_openssl` switch when you compile.

To install OpenSSL on non-Windows systems:

```bash
# macOS:
brew install openssl

# Debian/Ubuntu:
sudo apt install libssl-dev

# Arch/Manjaro:
openssl is installed by default

# Fedora:
sudo dnf install openssl-devel
```

On Windows, OpenSSL is simply hard to get working correctly.
The instructions
[here](https://tecadmin.net/install-openssl-on-windows/)
may (or may not) help.

## V sync

V's `sync` module and channel implementation uses libatomic.
It is most likely already installed on your system, but if not,
you can install it by doing the following:

```bash
macOS: already installed

# Debian/Ubuntu:
sudo apt install libatomic1

# Fedora/CentOS/RH:
sudo dnf install libatomic-static
```

> **Note**
> If you run into any trouble, or you have a different operating
> system or Linux distribution that doesn't install or work immediately, please see
> [Installation Issues](https://github.com/vlang/v/discussions/categories/installation-issues)
> and search for your OS and problem.
> If you cannot find your problem,
> please add it to an existing discussion if one exists for your OS,
> or create a new one if a main discussion does not yet exist for your OS.

And that is it! We are ready to start code in V!

In the next article, we will learn how to write our first "Hello, World" program in V.
