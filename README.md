用法
```rust
error_chain! {
    // The type defined for this error. These are the conventional
    // and recommended names, but they can be arbitrarily chosen.
    //
    // It is also possible to leave this section out entirely, or
    // leave it empty, and these names will be used automatically.
    types {
        Error, ErrorKind, ResultExt, Result;
    }

    // Without the `Result` wrapper:
    //
    // types {
    //     Error, ErrorKind, ResultExt;
    // }

    // Automatic conversions between this error chain and other
    // error chains. In this case, it will e.g. generate an
    // `ErrorKind` variant called `Another` which in turn contains
    // the `other_error::ErrorKind`, with conversions from
    // `other_error::Error`.
    //
    // Optionally, some attributes can be added to a variant.
    //
    // This section can be empty.
    links {
        Another(other_error::Error, other_error::ErrorKind) #[cfg(unix)];
    }

    // Automatic conversions between this error chain and other
    // error types not defined by the `error_chain!`. These will be
    // wrapped in a new error with, in the first case, the
    // `ErrorKind::Fmt` variant. The description and cause will
    // forward to the description and cause of the original error.
    //
    // Optionally, some attributes can be added to a variant.
    //
    // This section can be empty.
    foreign_links {
        Fmt(::std::fmt::Error);
        Io(::std::io::Error) #[cfg(unix)];
    }

    // Define additional `ErrorKind` variants.  Define custom responses with the
    // `description` and `display` calls.

    // 创建 ErrorKind
    errors {
        InvalidToolchainName(t: String) {
            description("invalid toolchain name")
            display("invalid toolchain name: '{}'", t)
        }

        // You can also add commas after description/display.
        // This may work better with some editor auto-indentation modes:
        UnknownToolchainVersion(v: String) {
            description("unknown toolchain version"), // note the ,
            display("unknown toolchain version: '{}'", v), // trailing comma is allowed
        }
    }
}

```

核心的两个链接: 
    * ! https://stevedonovan.github.io/rust-gentle-intro/6-error-handling.html
    * https://docs.rs/error-chain/0.12.1/error_chain/


首先需要理解 std::result::Result

```rust
pub enum Result<T, E> {
    /// Contains the success value
    Ok(T),
    /// Contains the error value
    Err(E),
}
```

然后需要理解:

```rust
// result_name, error_name 对应 error_chain macro 中的 types 中的名称
// 默认是Result, Error
// 这里定义了一个alias type
pub type $result_name<T> = ::std::result::Result<T, $error_name>;

// 然后又定义了 Error, 重点是里边的 kind 字段
#[derive(Debug)]
pub struct $error_name(
    // The members must be `pub` for `links`.
    /// The kind of the error.
    pub $error_kind_name,
    /// Contains the error chain and the backtrace.
    #[doc(hidden)]
    pub $crate::State,
);
// 继承了 ::std::error::Error
impl ::std::error::Error for $error_name {}

// 
```

理解foreign_links

```rust
error_chain!{
    foreign_links {
        Io(::std::io::Error);
    }
}

// 核心的 macro 代码
// link other Error type to error_chain Error type.

#[derive(Debug)]
pub enum $error_kind_name {

    /// A convenient variant for String.
    Msg(s: String) {
        description(&s)
        display("{}", s)
    }

    $(
        $(#[$meta_foreign_links])*
        $foreign_link_variant(err: $foreign_link_error_path) {
            description(::std::error::Error::description(err))
            display("{}", err)
        }
    ) *
}

// 会动态创建一个
ErrorKind::IO 到  `::std::io::Error` 的映射, 这里边的代码我就没看了, 太复杂了, 你可以理解
任何`::std::io::Error`都可以自动转换到生成的Error
```

理解 `ResultExt`, 这个对象对所有的 `Result` 对象进行了扩展, 支持了chain_err方法. 基本上你不会自己用到这个对象.

```
// chain error 内部会将 kind 做转换
/// Extends the error chain with a new entry.
pub fn chain_err<F, EK>(self, error: F) -> $error_name
    where F: FnOnce() -> EK, EK: Into<$error_kind_name> {
    $error_name::with_chain(self, Self::from_kind(error().into()))
}
```









# error-chain - Consistent error handling for Rust

[![Build Status](https://travis-ci.com/rust-lang-nursery/error-chain.svg?branch=master)](https://travis-ci.com/rust-lang-nursery/error-chain)
[![Latest Version](https://img.shields.io/crates/v/error-chain.svg)](https://crates.io/crates/error-chain)
[![License](https://img.shields.io/badge/license-MIT%2FApache--2.0-green.svg)](https://github.com/rust-lang-nursery/error-chain)

`error-chain` makes it easy to take full advantage of Rust's error
handling features without the overhead of maintaining boilerplate
error types and conversions. It implements an opinionated strategy for
defining your own error types, as well as conversions from others'
error types.

[Documentation (crates.io)](https://docs.rs/error-chain).

[Documentation (master)](https://rust-lang-nursery.github.io/error-chain).

## Quick start

If you just want to set up your new project with error-chain,
follow the [quickstart.rs] template, and read this [intro]
to error-chain.

[quickstart.rs]: https://github.com/rust-lang-nursery/error-chain/blob/master/examples/quickstart.rs
[intro]: http://brson.github.io/2016/11/30/starting-with-error-chain

## Supported Rust version

Please view the beginning of the [Travis configuration file](.travis.yml)
to see the oldest supported Rust version.

Note that `error-chain` supports older versions of Rust when built with
`default-features = false`.

## License

MIT/Apache-2.0
