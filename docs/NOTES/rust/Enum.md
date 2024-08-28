## 枚举的定义

```rust
enum IpAddrKind {
    V4,
    V6,
}
```

### 枚举值
可以像这样创建 `IpAddrKind` 两个不同成员的实例：
```rust
    let four = IpAddrKind::V4;
    let six = IpAddrKind::V6;
```

带有参数的枚举成员：
```rust
    enum IpAddr {
        V4(u8, u8, u8, u8),
        V6(String),
    }

    let home = IpAddr::V4(127, 0, 0, 1);

    let loopback = IpAddr::V6(String::from("::1"));
```
此时，每一个我们定义的枚举成员的名字也变成了一个构建枚举的实例的函数。也就是说，`IpAddr::V4()` 是一个获取 `String` 参数并返回 `IpAddr` 类型实例的函数调用。作为定义枚举的结果，这些构造函数会自动被定义。  
更复杂的枚举：
```rust
enum Message {
    Quit,
    Move { x: i32, y: i32 },
    Write(String),
    ChangeColor(i32, i32, i32),
}
```

### Option 枚举
`Option` 类型应用广泛因为它编码了一个非常普遍的场景，即一个值要么有值要么没值。  
标准库中的 `Option` 定义如下：
```rust
enum Option<T> {
    None,
    Some(T),
}
```
这段代码不能编译，因为它尝试将 Option<i8> 与 i8 相加：
```rust
    let x: i8 = 5;
    let y: Option<i8> = Some(5);

    let sum = x + y;
```
当在 Rust 中拥有一个像 i8 这样类型的值时，编译器确保它总是有一个有效的值。我们可以自信使用而无需做空值检查。只有当使用 Option<i8>（或者任何用到的类型）的时候需要担心可能没有值，而编译器会确保我们在使用值之前处理了为空的情况。

## match 控制流结构

基本用法：
```rust
enum Coin {
    Penny,
    Nickel,
    Dime,
    Quarter,
}

fn value_in_cents(coin: Coin) -> u8 {
    match coin {
        Coin::Penny => 1,
        Coin::Nickel => 5,
        Coin::Dime => 10,
        Coin::Quarter => 25,
    }
}
```

### 匹配 Option<T\>
```rust
fn main() {
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            None => None,
            Some(i) => Some(i + 1),
        }
    }

    let five = Some(5);
    let six = plus_one(five);
    let none = plus_one(None);
}
```

### 匹配是穷尽的
Rust 中的匹配是 穷尽的（exhaustive）：必须穷举到最后的可能性来使代码有效。  
例如，对于 `Option<T>` 类型，必须处理 `Some` 和 `None` 两种情况，缺一不可。下述写法是错误的：
```rust
    fn plus_one(x: Option<i32>) -> Option<i32> {
        match x {
            Some(i) => Some(i + 1),
        }
    }
```

### 通配模式和_占位符

#### 通配模式
```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        other => move_player(other),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn move_player(num_spaces: u8) {}
```

#### _占位符
```rust
    let dice_roll = 9;
    match dice_roll {
        3 => add_fancy_hat(),
        7 => remove_fancy_hat(),
        _ => reroll(),
    }

    fn add_fancy_hat() {}
    fn remove_fancy_hat() {}
    fn reroll() {}
```

## if let 简洁控制流
换句话说，可以认为 if let 是 match 的一个语法糖，它当值匹配某一模式时执行代码而忽略所有其他值。
```rust
    let config_max = Some(3u8);
    if let Some(max) = config_max {
        println!("The maximum is configured to be {}", max);
    }
```