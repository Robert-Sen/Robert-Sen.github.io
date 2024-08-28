## 包和 Crate

### crate
- crate 是 Rust 在编译时最小的代码单位。
- 单个 `.rs` 文件可以看作一个 crate 。
- crate 可以包含模块，模块可以定义在其他文件，然后和 crate 一起编译。  

crate 有两类：   

1. 二进制项  
  二进制项 可以被编译为可执行程序，比如一个命令行程序或者一个服务器。它们必须有一个 `main` 函数来定义当程序被执行的时候所需要做的事情。
1. 库  
  库 并没有 main 函数，它们也不会编译为可执行程序，它们提供一些诸如函数之类的东西，使其他项目也能使用这些东西。

crate root 是一个源文件，Rust 编译器以它为起始点，并构成你的 crate 的根模块  

### package
- 包（package） 是提供一系列功能的一个或者多个 crate。一个包会包含一个 Cargo.toml 文件，阐述如何去构建这些 crate。Cargo 就是一个包含构建你代码的二进制项的包。
- 包中至少包含一个 crate, 但至多包含一个库 crate(library crate)。

当我们执行如下命令时：
```
$ cargo new my-project
     Created binary (application) `my-project` package
$ ls my-project
Cargo.toml
src
$ ls my-project/src
main.rs
```
在此，我们有了一个只包含 src/main.rs 的包，意味着它只含有一个名为 my-project 的二进制 crate。  
如果一个包同时含有 src/main.rs 和 src/lib.rs，则它有两个 crate：一个二进制的和一个库的，且名字都与包相同。  
通过将文件放在 src/bin 目录下，一个包可以拥有多个二进制 crate：每个 src/bin 下的文件都会被编译成一个独立的二进制 crate。

## 定义模块来控制作用域与私有性

### 模块规则
- 从crate根节点开始: 当编译一个crate, 编译器首先在crate根文件（通常，对于一个库crate而言是src/lib.rs，对于一个二进制crate而言是src/main.rs）中寻找需要被编译的代码。
- 声明模块: 在crate根文件中，你可以声明一个新模块；比如，你用mod garden声明了一个叫做garden的模块。编译器会在下列路径中寻找模块代码：
    - 内联，在大括号中，当mod garden后方不是一个分号而是一个大括号
    - 在文件 src/garden.rs
    - 在文件 src/garden/mod.rs
- 声明子模块: 在除了crate根节点以外的其他文件中，你可以定义子模块。比如，你可能在src/garden.rs中定义了mod vegetables;。编译器会在以父模块命名的目录中寻找子模块代码：
    - 内联, 在大括号中，当mod vegetables后方不是一个分号而是一个大括号
    - 在文件 src/garden/vegetables.rs
    - 在文件 src/garden/vegetables/mod.rs
- 模块中的代码路径: 一旦一个模块是你crate的一部分， 你可以在隐私规则允许的前提下，从同一个crate内的任意地方，通过代码路径引用该模块的代码。举例而言，一个garden vegetables模块下的Asparagus类型可以在crate::garden::vegetables::Asparagus被找到。
- 私有 vs 公用: 一个模块里的代码默认对其父模块私有。为了使一个模块公用，应当在声明时使用pub mod替代mod。为了使一个公用模块内部的成员公用，应当在声明前使用pub。
- use 关键字: 在一个作用域内，use关键字创建了一个成员的快捷方式，用来减少长路径的重复。在任何可以引用crate::garden::vegetables::Asparagus的作用域, 你可以通过 use crate::garden::vegetables::Asparagus;创建一个快捷方式，然后你就可以在作用域中只写Asparagus来使用该类型。

下面创建一个名为 `backyard` 的 crate 来说明这些规则。  
这是文件目录树：  
```
backyard
├── Cargo.lock
├── Cargo.toml
└── src
    ├── garden
    │   └── vegetables.rs
    ├── garden.rs
    └── main.rs

```
文件名：src/main.rs
```rust
use crate::garden::vegetables::Asparagus;

pub mod garden;

fn main() {
  let plant = Asparagus {
    weigth: 1.2,
  };
  println!("I'm growing {:?}!", plant);
}
```
文件名：src/garden.rs
```rust
pub mod vegetables;
```
文件名：src/garden/vegetables.rs
```rust
#[derive(Debug)]
pub struct Asparagus {
  pub weigth: f64,
}
```

### 在模块中对相关代码进行分组
背景：  
在餐饮业，餐馆中会有一些地方被称之为 前台（front of house），还有另外一些地方被称之为 后台（back of house）。前台是招待顾客的地方，在这里，店主可以为顾客安排座位，服务员接受顾客下单和付款，调酒师会制作饮品。后台则是由厨师工作的厨房，洗碗工的工作地点，以及经理做行政工作的地方组成。  
文件名：src/lib.rs
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}

        fn seat_at_table() {}
    }

    mod serving {
        fn take_order() {}

        fn serve_order() {}

        fn take_payment() {}
    }
}
```

模块树：
```
crate
 └── front_of_house
     ├── hosting
     │   ├── add_to_waitlist
     │   └── seat_at_table
     └── serving
         ├── take_order
         ├── serve_order
         └── take_payment
```

## 引用模块项目的路径

路径有两种形式：

- 绝对路径（absolute path）从 crate 根开始，以 crate 名或者字面值 crate 开头。
- 相对路径（relative path）从当前模块开始，以 self、super 或当前模块的标识符开头。

例如：
文件名：src/lib.rs
```rust
mod front_of_house {
    mod hosting {
        fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

### 使用 pub 关键字暴露路径
上述代码无法通过编译，原因是模块 `front_of_house` 下的 `hosting` 模块默认是私有的，并且 `add_to_waitlist` 函数默认也是私有的，我们需要在其前面添加 `pub` 关键字，使其变成公有的。
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub fn eat_at_restaurant() {
    // 绝对路径
    crate::front_of_house::hosting::add_to_waitlist();

    // 相对路径
    front_of_house::hosting::add_to_waitlist();
}
```

### 使用 super 起始的关键字
类似于文件系统中以 `..` 开头的语法。

### 结构体和枚举
- 结构体中的成员默认私有。  
- 枚举中的成员默认共有。

## use 使用
`use` 关键字可以将指定路径引入作用域（scope），然后就可以直接调用路径中的项。  
使用 use 引入结构体、枚举和其他项时，习惯是指定它们的完整路径。例如：
```rust
use std::collections::HashMap;

fn main() {
    let mut map = HashMap::new();
    map.insert(1, 2);
}
```

### 使用 as 关键字提供新的名称
```rust
use std::fmt::Result;
use std::io::Result as IoResult;

fn function1() -> Result {
    // --snip--
    Ok(())
}

fn function2() -> IoResult<()> {
    // --snip--
    Ok(())
}
```

### 使用 pub use 重导出名称
通过 pub use 使名称可从新作用域中被导入至任何代码。
```rust
mod front_of_house {
    pub mod hosting {
        pub fn add_to_waitlist() {}
    }
}

pub use crate::front_of_house::hosting;

pub fn eat_at_restaurant() {
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
    hosting::add_to_waitlist();
}
```

### 嵌套路径来消除大量的 use 行
```rust
// --snip--
use std::{cmp::Ordering, io};
// --snip--
```

### glob 运算符
如果希望将一个路径下 所有 公有项引入作用域，可以指定路径后跟 *，glob 运算符：
```rust
use std::collections::*;
```