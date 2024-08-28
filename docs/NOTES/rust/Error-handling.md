## panic! 宏
当程序出现 bug 时，可以使用 `panic!` 宏，当执行这个宏时，程序会打印出一个错误信息，展开并清理栈数据，然后接着退出。  

当出现 panic 时，程序默认会开始 **展开**（unwinding），这意味着 Rust 会回溯栈并清理它遇到的每一个函数的数据，不过这个回溯并清理的过程有很多工作。另一种选择是直接 **终止**（abort），这会不清理数据就退出程序。那么程序所使用的内存需要由操作系统来清理。例如，如果你想要在release模式中 panic 时直接终止：
```rust
// Cargo.toml
[profile.release]
panic = 'abort'
```

### 使用 panic! 的 backtrace
有以下代码：
```rust
fn main() {
    let v = vec![1, 2, 3];

    v[99];
}
```
尝试运行：
```
$ cargo run
   Compiling panic v0.1.0 (file:///projects/panic)
    Finished dev [unoptimized + debuginfo] target(s) in 0.27s
     Running `target/debug/panic`
thread 'main' panicked at 'index out of bounds: the len is 3 but the index is 99', libcore/slice/mod.rs:2448:10
note: Run with `RUST_BACKTRACE=1` for a backtrace.
```
使用 backtrace：
```rust
$ RUST_BACKTRACE=1 cargo run
...
```

## Result 类型

`Result` 类型在标准库中的定义如下：
```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}
```

### 使用 `Result` 类型处理可恢复错误
```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => {
            panic!("Problem opening the file: {:?}", error)
        },
    };
}
```

### 匹配不同类型的错误
```rust
use std::fs::File;
use std::io::ErrorKind;

fn main() {
    let f = File::open("hello.txt");

    let f = match f {
        Ok(file) => file,
        Err(error) => match error.kind() {
            ErrorKind::NotFound => match File::create("hello.txt") {
                Ok(fc) => fc,
                Err(e) => panic!("Problem creating the file: {:?}", e),
            },
            other_error => panic!("Problem opening the file: {:?}", other_error),
        },
    };
}
```

### unwrap 和 expect
```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").unwrap();
}
```

```rust
use std::fs::File;

fn main() {
    let f = File::open("hello.txt").expect("Failed to open hello.txt");
}
```

### 传播错误
当编写一个其实现会调用一些可能会失败的操作的函数时，除了在这个函数中处理错误外，还可以选择让调用者知道这个错误并决定该如何处理。这被称为 **传播**（propagating）错误。例如：
```rust

#![allow(unused_variables)]
fn main() {
use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let f = File::open("hello.txt");

    let mut f = match f {
        Ok(file) => file,
        Err(e) => return Err(e),
    };

    let mut s = String::new();

    match f.read_to_string(&mut s) {
        Ok(_) => Ok(s),
        Err(e) => Err(e),
    }
}
}
```

### ? 运算符
```rust

use std::io;
use std::io::Read;
use std::fs::File;

fn read_username_from_file() -> Result<String, io::Error> {
    let mut f = File::open("hello.txt")?;
    let mut s = String::new();
    f.read_to_string(&mut s)?;
    Ok(s)
}
```
它实现了和上个例子相同的功能。

### main 函数返回 Result
```rust
use std::error::Error;
use std::fs::File;

fn main() -> Result<(), Box<dyn Error>> {
    let f = File::open("hello.txt")?;

    Ok(())
}
```