## 所有权

### 所有权规则
1. Rust 中的每一个值都有一个 所有者（owner）。
2. 值在任一时刻有且只有一个所有者。
3. 当所有者（变量）离开作用域，这个值将被丢弃。

### 变量作用域
```rust
fn main() {
    {                      // s 在这里无效, 它尚未声明
        let s = "hello";   // 从此处起，s 是有效的
        // 使用 s
    }                      // 此作用域已结束，s 不再有效
}
```
变量 `s` 绑定到了一个字符串字面值，这个字符串值是硬编码进程序代码中的。这个变量从声明的点开始直到当前 作用域 结束时都是有效的。

### String 类型  
基于字符串字面值创建一个 String 变量，如下：
```rust
let s = String::from("hello");
```

#### 内存与分配
其内存分配在堆上，所以能够存储在编译时未知大小的文本。
```rust
fn main() {
    {
        let s = String::from("hello"); // 从此处起，s 是有效的
        // 使用 s
    }                                  // 此作用域已结束，
                                       // s 不再有效
}
```

### 移动 (move)
```rust
let s1 = String::from("hello");
let s2 = s1;
```
在 `let s2 = s1` 中，先将 s1 在内存中的值的指针拿给 s2，然后 s1 不再有效，因此 Rust 不需要在 s1 离开作用域后清理任何东西。 
遵循 `所有权规则` 中的第二条

### 克隆 (clone)
```rust
let s1 = String::from("hello");
let s2 = s1.clone();
```
在 `let s2 = s1.clone()` 中，s2 会克隆一份 s1 的值，此后 s1 和 s2 都是有效的。

### 拷贝 (copy)
只在栈上的数据可以快速拷贝。即在变量赋值时不会发生移动。

### 所有权与函数
```rust
fn main() {
  let s1 = String::from("hello");   // s1 进入作用域
  let s2 = pass_ownership(s1);     // s1 的值移动到函数里，然后又从函数内返回移动给 s2
                                    // s1 到这里不再有效
}
fn pass_ownership(_str: String) -> String{
  _str    // _str 的值移动到函数外
}
```

## 引用与借用

```rust
fn main() {
  let s1 = String::from("hello");
  let len = length(&s1);
  println!("Length of '{}': {}", s1, len);
}
fn length(s: &String) -> usize {
  s.len()
}
```
使用 **引用** (reference) 作为参数时不会发生所有权的转移。  
Rust 将创建一个引用的行为称为 **借用** (borrowing)  
默认引用不允许修改引用的值

### 可变引用
```rust
fn main() {
    let mut s = String::from("hello");

    change(&mut s);
}
fn change(some_string: &mut String) {
    some_string.push_str(", world");
}
```

### 引用的规则

- 在任意给定时间，要么 只能有一个可变引用，要么 只能有多个不可变引用。
- 引用必须总是有效的

## Slice 类型
slice 允许你引用集合中一段连续的元素序列，而不用引用整个集合。**slice 是一类引用**，所以它没有所有权。  
考虑下面一个程序：  
```rust
// 返回一个以空格分隔的字符串的第一个单词的最后一个字母的索引
fn first_word(s: &String) -> usize {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return i;
        }
    }

    s.len()
}

fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s); // word 的值为 5

    s.clear(); // 这清空了字符串，使其等于 ""

    // word 在此处的值仍然是 5，
    // 但是没有更多的字符串让我们可以有效地应用数值 5。word 的值现在完全无效！
}
```
可以用 `字符串 slice` 解决

### 字符串 slice
```rust
let s = String::from("hello world");

let slice1 = &s[..];    // 整个切片
let slice2 = &s[3..];   // 从第3个字符开始到最后
let slice3 = &s[..3];   // 从第0个字符到第3个字符
let slice4 = &s[3..5];  // 从第3个字符到第5个字符
```

我们尝试使用字符串 slice 来解决上面的问题  
```rust
fn first_word(s: &String) -> &str {
    let bytes = s.as_bytes();

    for (i, &item) in bytes.iter().enumerate() {
        if item == b' ' {
            return &s[0..i];
        }
    }

    &s[..]
}

fn main() {
    let mut s = String::from("hello world");

    let word = first_word(&s);

    s.clear(); // 错误!

    println!("the first word is: {}", word);
}

```

### 其他类型的 slice
```rust
fn main() {
  let a = [1, 2, 3, 4, 5];

  let slice = &a[1..3];
}
```