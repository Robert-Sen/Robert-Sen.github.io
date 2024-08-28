## 结构体定义和实例化

### 定义
```rust
struct User {
    active: bool,
    username: String,
    email: String,
    sign_in_count: u64,
}
```
结构体中由数据的名字和类型组成，我们称为 **字段** (field)。

### 实例化
```rust
fn main() {
    let mut user1 = User {
        email: String::from("someone@example.com"),
        username: String::from("someusername123"),
        active: true,
        sign_in_count: 1,
    };
    let email = user1.email;  // 使用 '.' 访问结构体中的字段
}
```
注意，rust并不允许只将单个字段标记为可变，必须整个示例是可变的。

#### 使用字段初始化简写语法
```rust
fn build_user(email: String, username: String) -> User {
    User {
        email,      // 字段 email 和参数 email 相同，可以简写
        username,
        active: true,
        sign_in_count: 1,
    }
}
```

#### 使用结构体更新语法实例
不使用结构体更新语法：
```rust
fn main() {
    let user2 = User {
        active: user1.active,
        username: user1.username,
        email: String::from("another@example.com"),
        sign_in_count: user1.sign_in_count,
    };
}
```
使用结构体更新语法：
```rust
fn main() {
    let user2 = User {
        email: String::from("another@example.com"),
        ..user1
    };
}
```
注意，结构体更新语法会发生是所有权转移。

### 元组结构体
元组结构体 (tuple structs) 没有字段名。
```rust
struct Color(i32, i32, i32);
struct Point(i32, i32, i32);

fn main() {
    let black = Color(0, 0, 0);
    let origin = Point(0, 0, 0);
}
```

### 类单元结构体
类单元结构体 (unit-like structs) 没有任何字段。  
类单元结构体常常在你想要在某个类型上实现 trait 但不需要在类型中存储数据的时候发挥作用。
```rust
struct AlwaysEqual;

fn main() {
    let subject = AlwaysEqual;
}
```

## 结构体示例

```rust
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!(
        "The area of the rectangle is {} square pixels.",
        area(&rect1)
    );
}

fn area(rectangle: &Rectangle) -> u32 {
    rectangle.width * rectangle.height
}
```

### 通过派生 trait 增加功能
```rust
  -- snip --
  println!("rect1 is {}", rect1);   // 尝试打印 rect1，编译报错
```
正确打印方法如下：
```rust
#[derive(Debug)]
struct Rectangle {
    width: u32,
    height: u32,
}

fn main() {
    let rect1 = Rectangle {
        width: 30,
        height: 50,
    };

    println!("rect1 is {:?}", rect1);
}
```
在 `{}` 中加入 `:?` 指示符告诉 `println!` 我们想要使用叫做 `Debug` 的输出格式。  
在结构体定义之前加上外部属性 `#[derive(Debug)]`，以此为结构体显式选择打印出调试信息的功能。  
还可以使用 `{:#?}` 替换 `println!` 字符串中的 `{:?}`，打印结果如下：
```
$ cargo run
   Compiling rectangles v0.1.0 (file:///projects/rectangles)
    Finished dev [unoptimized + debuginfo] target(s) in 0.48s
     Running `target/debug/rectangles`
rect1 is Rectangle {
    width: 30,
    height: 50,
}
```

还可以使用 `dbg!` 宏来打印数值如：`dbg!(&rect1)`。

## 方法

### 定义方法
```rust
struct Rectangle {
    width: u32,
    height: u32,
}

impl Rectangle {
    fn area(&self) -> u32 {
        self.width * self.height
    }
}
```
在 `area` 的签名中，使用 `&self` 来替代 `rectangle: &Rectangle`，`&self` 实际上是 `self: &Self` 的缩写。在一个 `impl` 块中，`Self` 类型是 `impl` 块的类型的别名。方法的第一个参数必须有一个名为 self 的Self 类型的参数。  
通常，但并不总是如此，与字段同名的方法将被定义为只返回字段中的值，而不做其他事情。这样的方法被称为 getters。例如：
```rust
-- snip --
impl Rectangle {
  -- snip --
  fn width() -> u32 {
    width
  }
}
```

### 关联函数
所有在 `impl` 块中定义的函数被称为 **关联函数** (associated functions)，因为它们与 `impl` 后面的类型相关。  
我们可以定义不以 self 为第一参数的关联函数，例如 `String::from` 函数。
```rust
impl Rectangle {
    fn square(size: u32) -> Self {
        Self {
            width: size,
            height: size,
        }
    }
}
fn main() {
  let sq = Rectangle::square(3);
}
```

### 多个 impl 块
每个结构体都允许拥有多个 `impl` 块。