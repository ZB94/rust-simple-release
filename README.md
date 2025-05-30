# Rust simple release

English | [简体中文](./docs/README.zh-CN.md)

**Extremely simple all-in-one** release for your rust project!

This CI is for small and middle rust project, which doesn't need complex CI/CD. It supports rust workspace, but only allows build and upload from one package. (request features if you want more packages to build :) And it supports building multi features, bins and lib.

**You don't need to worry about openssl deps, this action will solve it.** This action will automatically detect openssl, and add `vendored` feature.

If you are using nightly or other channels, please add it to `rust-toolchain.toml`.

This action will automatically setup rust dev environment, and other tools like `cargo-zigbuild`. **It doesn't use containers**, so if you have other dependencies, just install and config them before running this action.

## Usage

Please set your Github token before using this action. The example below uses `GH_TOKEN` as the secret name, you can change it to your own secret name.

### Simple

assume that your project only has only one package with no features. This can release bins if your package has bins, or release a lib if your package is a lib package.

```yaml
name: rust release action
on:
  push:
    tags:
      - "v*"
jobs:
  release:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v4
      - name: test build rust project
        uses: lxl66566/rust-simple-release@main
        with:
          targets: |
            aarch64-unknown-linux-gnu
            aarch64-unknown-linux-musl
            x86_64-pc-windows-msvc
            x86_64-unknown-linux-musl
            x86_64-unknown-linux-gnu
            aarch64-apple-darwin
            x86_64-apple-darwin

          token: ${{ secrets.GH_TOKEN }}
```

### Full

```yaml
name: test rust release action
on:
  push:
    tags:
      - "v*"
jobs:
  basic_test:
    runs-on: ${{ matrix.os }}
    strategy:
      matrix:
        os: [ubuntu-latest, windows-latest, macos-latest]
    steps:
      - uses: actions/checkout@v4
      - name: test build rust project
        uses: lxl66566/rust-simple-release@main
        with:
          # Targets to compile, seperated by comma or newline
          # Support Linux, Windows and Darwin
          targets: |
            aarch64-unknown-linux-gnu
            aarch64-unknown-linux-musl
            x86_64-pc-windows-msvc
            x86_64-unknown-linux-musl
            x86_64-unknown-linux-gnu
            aarch64-apple-darwin
            x86_64-apple-darwin

          # Choose one package to build. If not set, it will build first package in workspace.
          package: openssl-test

          # whether to build lib in the package. If the package has both lib and bin targets, you need to set this option to build lib, otherwise the lib will be ignored, only build bins.
          # If the package has lib target and no bin targets, it will build lib by default.
          # If the package has no lib target, set this option to true will cause an error.
          lib: true

          # Choose bins to build, seperated by comma. If not set, it will build all bins in the package.
          # This `bins` option should be a subset of target bins in `Cargo.toml`.
          bins: my-action-test, my-action-test2

          # Features to enable, seperated by comma (allow space)
          features: test1, test2

          # Files or folders to pack into release assets, relative path seperated by comma.
          # The files and folers will be added to the root path of archive.
          # Build outputs (bins and lib) will automatically added to the archive, you don't need to add them twice.
          files_to_pack: README.md, LICENSE, assets

          # release create options, see https://cli.github.com/manual/gh_release_create
          release_options: --draft --title 123

          # GITHUB TOKEN, **REQUIRED**
          token: ${{ secrets.GH_TOKEN }}

        env:
          # debug level, print more logs
          debug: 1
```

## Hint

- Now the archive format is not choosable: `zip` for windows and `tar.gz` for other systems. (request a feature if you want to change it :)

## License

MIT
