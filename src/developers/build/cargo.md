# Cargo

If you are comfortable with installing all dependencies on your host system,
you need to install the following software:

* LLVM
* bpftool
* Rust, Cargo

## LLVM

We need a recent version of LLVM (at least 12) to build BPF programs.

LLVM has an official [apt repository](https://apt.llvm.org/) with recent
stable versions.

Distributions with up to date software repositories like Arch, Fedora, openSUSE
Tumbleweed are shipping recent versions of LLVM.

In more stable and not up to date distributions (CentOS, openSUSE Leap, RHEL,
SLES), using some kind of development repository might be an option. For
example, openSUSE Leap users can use the following devel repo:

```bash
zypper ar -r -p 90 https://download.opensuse.org/repositories/devel:/tools:/compiler/openSUSE_Leap_15.3/devel:tools:compiler.repo
zypper ref
zypper up --allow-vendor-change
zypper in clang llvm
```

If there is no packaging of recent LLVM versions for your distribution, there
is also an option to [download binaries](https://releases.llvm.org/download.html).

## bpftool

bpftool is the official CLI for interacting with BPF subsystem.

Distributions with up to date software (Arch, Fedora, openSUSE Tumbleweed)
usually provide packaging for it.

Especially for more stable and less up to date distributions, but even
generally, we would recommend to build bpftool from source. Both of
them are the part of the Linux kernel source.

bpftool is available in its own repo - [github.com/libbpf/bpftool](https://github.com/libbpf/bpftool).
It lets you to build bpftool with the following commands:

```bash
git clone --recurse-submodules https://github.com/libbpf/bpftool.git
cd src
make
sudo make install prefix=/usr
```

## Installing Rust

Our recommended way of installing Rust is using **rustup**.
[Their website](https://rustup.rs/) contains installation instruction.

After installing rustup, let's install lint tools:

```bash
rustup component add clippy rustfmt
```

And then cargo-libbpf, needed for building the BPF part:

```bash
cargo install libbpf-cargo
```

## Building lockc

After installing all needed dependencies, it's time to build lockc.

The build of the project can be done with:

```bash
cargo build
```

Running tests:

```bash
cargo test
```

Running lints:

```bash
cargo clippy
```

## Installing lockc

To install lockc on your host, use the following command:

```bash
cargo xtask install
```

Do not run this command with sudo! Why?

tl;dr: you will be asked for password when necessary, don't worry!

Explanation: Running cargo with sudo ends with weird consequences like not
seing cargo content from your home directory or leaving some files owned by
root in `target`. When any destination directory is owned by root, sudo will
be launched automatically by `xtask install` just to perform necessary
installation steps.

By default it tries to install lockcd binary in `/usr/local/bin`, but the
destination directory can be changed by the following arguments:

* `--destdir` - the rootfs of your system, default: `/`
* `--prefix` - prefix of the most of installation destinations, default:
  `usr/local`
* `--bindir` - directory for binary files, default: `bin`
* `--unitdir` - directory for systemd units, default: `lib/systemd/system`
* `--sysconfdir` - directory for configuration files, default: `etc`

By default, binaries are installed from the `debug` target profile. If you want
to change it, use the `--profile` argument. `--profile release` is what you
most likely want to use when packaging or installing on the production system.

## Building tarball with binary and unit

To make distribution of lockc for Docker users easier, we have a possibility of
building an archive with binary and systemd unit which can be just unpacked in
`/` directory. It can be done by the following command:

```bash
cargo xtask bintar
```

By default it archives lockcd binary in `usr/local/bin`, but the
destination directory can be changed by the following arguments:

* `--prefix` - prefix of the most of installation destinations, default:
  `usr/local`
* `--bindir` - directory for binary files, default: `bin`
* `--unitdir` - directory for systemd units, default: `lib/systemd/system`
* `--sysconfdir` - directory for configuration files, default: `etc`

By default, binaries are installed from the `debug` target profile. If you want
to change it, use the `--profile` argument. `--profile release` is what you
most likely want to use when creating a tarball for releases and production
systems.

The resulting binary should be available as `target/[profile]/lockc.tar.gz`
(i.e. `target/debug/lockc.tar.gz`).
