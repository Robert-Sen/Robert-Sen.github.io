## 变量

### 不可变变量
```rust
let x = 5;
x = 6;  // 编译报错
```

### 可变变量
```rust
let mut x = 5;
x = 6;  // 编译通过
```

### 常量
```rust
const PI: f64 = 3.14;
```
rust 中规定常量命名全部用大写字母加下划线作为分隔号。

### 隐藏
```rust
fn main() {
    let x = 5;

    let x = x + 1;  // 这个 x 会覆盖上一个 x

    {
        let x = x * 2;
        println!("The value of x in the inner scope is: {x}");  // 输出12
    } // 上面一个 x 的作用域在这里结束

    println!("The value of x is: {x}"); // 输出6
}
```

## 数据类型

rust 中有两类数据类型：标量 (scalar) 和复合 (compound)。
### 标量类型
- **整型**  
  有符号：i8 ~ i128  
  无符号：u8 ~ u128
  另外还有两个特殊类型 `isize` 和 `usize`, 它们依赖运行程序的计算机架构：64 位架构上它们是 64 位的， 32 位架构上它们是 32 位的。  
  默认类型为 `i32`
- **浮点型**  
  两类：`f32` 和 `f64`。  
  默认类型为 `f64`
- **布尔型**  
  `true` 和 `false`
- **字符型**  
  `char`关键字  
  char 类型的大小为**四个字节**，代表一个 Unicode 标量值。

### 复合类型
- 元组 (tuple)
  ```rust
  fn tuple() {
    let tup1: (&str, i32, f64) = ("LiHua", 20, 125.5);  // 元组的长度在创建时必须确定
    let (name, age, weight) = tup1; // 使用模式匹配解构元组值
    println!("name: {}, age: {}, weight: {}", name, age, weight);
    println!("name: {}, age: {}, weight: {}", tup1.0, tup1.1, tup1.2);    
  }
  ```
  不带任何值的元组叫做 **单元 (uinit)**, 写法为`()`
- 数组 (array)  
  数组中的元素的类型必须相同，且长度在创建数组时确定，在栈上分配。
  ```rust
  fn array() {
    let a = [1, 2, 3, 4, 5];  // 自动推导类型和长度
    let b: [i32; 5] = [1, 2, 3, 4, 5];  // 显示说明类型和长度
    let c = [3; 5];   // 长度为5， 元素的初值为3
    
    let a1 = a[1];  // 访问数组 a 的第1个元素，也就是2
    let b2 = b[2];
  }
  ```

## 函数
```rust
fn max(a: i32, b: i32) -> i32 {
  if a > b {
    a
  } else {
    b
  }
}
```

### 参数
每个参数由两部分组成：参数名和参数类型

### 语句和表达式
函数体由一系列语句和一个可选的结尾表达式构成。

- 语句 (Statements)  
  执行一些操作，但无返回值。例如：
  ```rust
  fn main() {
    let y = 6;
  }
  ```
  其中函数定义本身是一个语句，`main` 函数里还有一个赋值语句。  
  因此，rust 中不能连续赋值。

- 表达式 (Expressions)
  具有返回值。包括：  
    - 数学运算
    - 函数调用
    - 宏调用
    - 用大括号创建的作用域

### 返回值  
  如上 `max` 函数中，在 '->' 后声明了一个 i32 类型的返回值。

## 注释

### 一般注释
```rust
// This is single-line comment.
/* This is multi-line comment. */
fn main() {
  println!("hello world");
}
```

### 文档注释
```rust
/// This is document single-line comment.
/** 
  This is document multi-line comment.
*/
fn main() {
  println!("hello world");
}
```
注意:   

- 文档注释需要位于 lib 类型的包中，例如 src/lib.rs 中  
- 文档注释可以使用 markdown语法！例如 # Examples 的标题，以及代码块高亮  
- 被注释的对象需要使用 pub 对外可见，记住：文档注释是给用户看的，内部实现细节不应该被暴露出去  

## 控制流

### if 表达式
```rust
fn is_even(x: i32) -> bool{
  if x % 2 == 0 {
    true
  } else if x % 2 != 0{
    false
  } else {
    println!("error");
  }
}
```

### 循环表达式
有三种循环：  

- loop  
  相当于 `while(true)` 循环  
  **循环标签**  
  ```rust
  let mut i = 0;
  let mut j = 0;
  'break_tag: loop {
    i += 1;
    loop {
      if i == 5 {
        break 'break_tag;
      }
      j += 1;
      if j % 5 == 0 {
        break;
      }
    }
  }
  ```
- while  
  ```rust
  let mut i = 0;
  while i != 100 {
    println!("{i}");
    i += 1;
  }
  ```
- for  
  ```rust
  fn main() {
    // 打印结果为：
    // 3!
    // 2!
    // 1!
    for n in (1..4).rev() {
      println!("{n}!");
    }
  }
  ```
  