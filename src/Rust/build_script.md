
# 构建你的Rust编译版本信息
有时，我们构建的`Rust`二进制包需要交付给它人使用。难免存在需要追溯当前二进制包的版本信息的问题。包括构建的包`系统环境(OS)`,`Rust版本`，`代码分支`，`commit_id`等常用信息。
有了这些信息，不管是定位问题还是复现`bug`,都清晰很多。

## 思考
通过获取环境变量或外部参数，将信息编译统一打包到编译文件中。好在Rust提供了`build script`的功能，详见链接：[build-scripts.html](https://doc.rust-lang.org/stable/rust-by-example/cargo/build_scripts.html).
因此我们可以通过这种方式将编译信息生成出对应的`rust`代码,通过`Rust`提供的`include!`宏来讲生成代码和目标代码打包。我们就可以在项目中使用我们生成的静态信息了。

## 使用 shadow-rs
[https://github.com/baoyachi/shadow-rs](https://github.com/baoyachi/shadow-rs)


获取环境变量，获取外部参数，判断当前构建环境，直到生成代码。这一系列过程，本身不复杂，需要判断条件，索性就写了一个构建脚本的工具，使用如下

## step1 
在当前项目目录下的`Cargo.toml`文件，配置如下信息
```toml
[package]
build = "build.rs"

[build-dependencies]
shadow-rs = "0.3"
```

## step 2
在当前项目的`build.rs`文件下，配置如下：
```rust
fn main() -> shadow_rs::SdResult<()> {
    let src_path = std::env::var("CARGO_MANIFEST_DIR")?;
    let out_path = std::env::var("OUT_DIR")?;
    shadow_rs::Shadow::build(src_path, out_path)?;
    Ok(())
}
```

## step 3
在当前项目下找到`bin`文件，通常是`main.rs`文件，你可以在`Cargo.toml`的`[bin]`配置找到当前主文件，然后添加如下信息
```rust
pub mod shadow{
    include!(concat!(env!("OUT_DIR"), "/shadow.rs"));
}
```

## step 4
然后你就可以使用`shadow`的常量信息了,大概如下
```rust
fn main() {
    println!("{}",shadow::BRANCH); //master
    println!("{}",shadow::COMMIT_HASH);//8405e28e64080a09525a6cf1b07c22fcaf71a5c5
    println!("{}",shadow::SHORT_COMMIT);//8405e28e
    println!("{}",shadow::COMMIT_DATE);//2020-08-16T06:22:24+00:00
    println!("{}",shadow::COMMIT_AUTHOR);//baoyachi
    println!("{}",shadow::COMMIT_EMAIL);//xxx@gmail.com

    println!("{}",shadow::BUILD_OS);//macos-x86_64
    println!("{}",shadow::RUST_VERSION);//rustc 1.45.0 (5c1f21c3b 2020-07-13)
    println!("{}",shadow::RUST_CHANNEL);//stable-x86_64-apple-darwin (default)
    println!("{}",shadow::CARGO_VERSION);//cargo 1.45.0 (744bd1fbb 2020-06-15)
    println!("{}",shadow::PKG_VERSION);//0.3.13
    println!("{}",shadow::CARGO_LOCK);

    println!("{}",shadow::PROJECT_NAME);//shadow-rs
    println!("{}",shadow::BUILD_TIME);//2020-08-16 14:50:25
    println!("{}",shadow::BUILD_RUST_CHANNEL);//debug

    ...
}
```

通常，你会使用`clap`的`crates`库.

详细使用，请参考链接例子：[https://github.com/baoyachi/shadow-rs/tree/master/example_shadow](https://github.com/baoyachi/shadow-rs/tree/master/example_shadow)




# 原文地址

