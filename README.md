<div align="center">

<img src="./resources/logo.svg" alt="lazydb" width="300">

lazydb is currently in alpha

A cross-platform TUI database management tool written in Rust



</div>

## Features

- Cross-platform support (macOS, Windows, Linux)
- Multiple Database support (MySQL, PostgreSQL, SQLite)
- Intuitive keyboard only control

## TODOs

- [ ] SQL editor
- [ ] Custom key bindings
- [ ] Custom theme settings
- [ ] Support the other databases

## What does "lazydb" come from?

lazydb is designed for developers who want a lazy, effortless way to manage databases. The name reflects the philosophy of making database management as simple and intuitive as possible.

## Installation

### With Cargo (Linux, macOS, Windows)

If you already have a Rust environment set up, you can use the `cargo install` command:

```
cargo install lazydb
```

### From source

Clone the repository and build from source:

```
git clone https://github.com/fujis/lazydb.git
cd lazydb
cargo build --release
```

## Usage

```
$ lazydb
```

```
$ lazydb -h
USAGE:
    lazydb [OPTIONS]

FLAGS:
    -h, --help       Prints help information
    -V, --version    Prints version information

OPTIONS:
    -c, --config-path <config-path>    Set the config file
```

If you want to add connections, you need to edit your config file. For more information, please see [Configuration](#Configuration).

## Keymap

| Key | Description |
| ---- | ---- |
| <kbd>h</kbd>, <kbd>j</kbd>, <kbd>k</kbd>, <kbd>l</kbd> | Scroll left/down/up/right |
| <kbd>Ctrl</kbd> + <kbd>u</kbd>, <kbd>Ctrl</kbd> + <kbd>d</kbd> | Scroll up/down multiple lines |
| <kbd>g</kbd> , <kbd>G</kbd> | Scroll to top/bottom |
| <kbd>H</kbd>, <kbd>J</kbd>, <kbd>K</kbd>, <kbd>L</kbd> | Extend selection by one cell left/down/up/right |
| <kbd>y</kbd> | Copy a cell value |
| <kbd>←</kbd>, <kbd>→</kbd> | Move focus to left/right |
| <kbd>c</kbd> | Move focus to connections |
| <kbd>/</kbd> | Filter |
| <kbd>?</kbd> | Help |
| <kbd>1</kbd>, <kbd>2</kbd>, <kbd>3</kbd>, <kbd>4</kbd>, <kbd>5</kbd> | Switch to records/columns/constraints/foreign keys/indexes tab |
| <kbd>Esc</kbd> | Hide pop up |

## Configuration

The location of the file depends on your OS:

- macOS: `$HOME/.config/lazydb/config.toml`
- Linux: `$HOME/.config/lazydb/config.toml`
- Windows: `%APPDATA%/lazydb/config.toml`

The following is a sample config.toml file:

```toml
[[conn]]
type = "mysql"
user = "root"
host = "localhost"
port = 3306

[[conn]]
type = "mysql"
user = "root"
host = "localhost"
port = 3306
password = "password"
database = "foo"
name = "mysql Foo DB"

[[conn]]
type = "postgres"
user = "root"
host = "localhost"
port = 5432
database = "bar"
name = "postgres Bar DB"

[[conn]]
type = "sqlite"
path = "/path/to/baz.db"
```

## Contribution

Contributions, issues and pull requests are welcome!
