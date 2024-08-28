Cargo 是 Rust 的构建系统和包管理器。可以用来构建代码、下载依赖库，以及编译这些库。

### cargo new [<project\>]
  创建项目  
  创建 hello_cargo 项目: `cargo new hello_cargo`

### cargo build
  构建项目  
  使用 `cargo build --release` 进行编译优化

### cargo run
  构建并运行项目

### cargo check
  快速检查代码是否可编译，但不产生可执行文件

### Cargo.toml
  TOML(Tom's Obvious, minimal Language)  
  [package]: 配置包的表块  
  [dependencies]: 配置依赖的表块

### Cargo.lock
  确保项目在构建时只会使用指定的依赖版本 

### cargo update
  升级 crate(代码包) 