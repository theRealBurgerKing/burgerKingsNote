# Error handling
## 什么是哨兵值？

哨兵值是一种特殊的值，用来表示"无效"、"结束"或"特殊状态"。在编程中，它常用于标记数据的边界或特殊情况。

## 幻灯片提到的三种语言中的哨兵值：

### **C 语言**

- 字符串以 `\0`（空字符）作为结束标记
- 指针可以用 `NULL` 表示"无效指针"

### **Go 语言**

- 使用 `nil` 表示空指针、空接口、空切片等
- 支持多返回值，常用 `(value, error)` 模式处理错误

### **Pascal 语言**

- 较老的语言，也使用特殊值标记无效状态
- 例如使用特定数值表示"未初始化"或"无效"

## Rust 的不同之处

Rust 语言通过**类型系统**避免了传统哨兵值的问题：

- 使用 `Option<T>` 类型明确表示"可能有值，也可能没有"
    - `Some(value)` 表示有值
    - `None` 表示无值
- 使用 `Result<T, E>` 处理可能失败的操作
    - `Ok(value)` 表示成功
    - `Err(error)` 表示错误

这样做的好处是：编译器会强制你处理"无值"的情况，避免了像 C 语言中忘记检查 `NULL` 导致的程序崩溃。



## 第一张：Checked Exceptions（受检异常）

**只有 Java** 使用受检异常机制。

### 什么是受检异常？

- 编译器**强制**要求你处理的异常
- 必须用 `try-catch` 捕获，或在方法签名中用 `throws` 声明
- 例如：`IOException`、`SQLException`

java

```java
// 必须处理，否则无法编译
public void readFile() throws IOException {
    FileReader file = new FileReader("file.txt");
}
```

## 第二张：Unchecked Exceptions（非受检异常）

大多数现代语言都使用非受检异常：

- **Java**（也支持，如 `RuntimeException`）
- **C++**
- **C#**
- **Python**
- **Swift**
- **Kotlin**
- **ML 语言家族**（如 OCaml、F#）
- **Rust？**（用问号表示疑问）

### 什么是非受检异常？

- 编译器**不强制**要求处理
- 程序员可以选择是否捕获
- 如果不处理，异常会向上传播直到程序崩溃

## Rust 的特殊之处（"...Rust?"）

Rust **没有传统的异常机制**！幻灯片用问号表示 Rust 的独特性：

### Rust 的错误处理方式：

1. **可恢复错误**：使用 `Result<T, E>` 类型

rust

```rust
   fn read_file() -> Result<String, io::Error> {
       std::fs::read_to_string("file.txt")
   }
```

2. **不可恢复错误**：使用 `panic!`

rust

```rust
   panic!("致命错误！");
```

### Rust 的优势：

- 没有"异常抛出"的隐藏控制流
- 通过类型系统明确标记可能失败的操作
- 编译器强制处理 `Result`，但方式更灵活（用 `?` 运算符传播错误）
- 避免了 Java 受检异常的繁琐，也避免了非受检异常被忽略的风险

这就是为什么 Rust 既不属于"受检异常"也不属于"非受检异常"阵营——它用类型系统彻底重新设计了错误处理！















这张幻灯片介绍了 **Sum types（和类型）**，这是函数式编程和现代类型系统中的重要概念。

## 什么是 Sum types？

Sum types（也叫**代数数据类型**、**枚举类型**、**标签联合**）允许一个值是**多种可能类型中的一种**。

### 形象理解：

- **Product types（积类型）**：同时包含多个值（如结构体/类）
    - 例：`struct Person { name: String, age: u32 }` 同时有姓名**和**年龄
- **Sum types（和类型）**：只能是多个选项中的**一个**
    - 例：`enum Status { Success, Error }` 要么成功**要么**失败

## 支持 Sum types 的语言

### **Rust**

rust

```rust
enum Result<T, E> {
    Ok(T),
    Err(E),
}

enum Option<T> {
    Some(T),
    None,
}
```

### **Swift**

swift

```swift
enum Result<Success, Failure: Error> {
    case success(Success)
    case failure(Failure)
}
```

### **Haskell**

haskell

```haskell
data Maybe a = Just a | Nothing
data Either a b = Left a | Right b
```

### **ML 语言家族**（OCaml、F#、SML）

ocaml

```ocaml
type 'a option = Some of 'a | None
```

### **Java（Recently）** - Java 17+ 的 Sealed Classes

java

```java
sealed interface Result<T, E> permits Success, Failure {
    record Success<T, E>(T value) implements Result<T, E> {}
    record Failure<T, E>(E error) implements Result<T, E> {}
}
```

## Sum types 的优势

1. **类型安全**：编译器确保你处理了所有可能的情况
2. **无需 null**：用 `Option`/`Maybe` 明确表示"可能没有值"
3. **清晰的错误处理**：`Result` 类型明确区分成功和失败
4. **模式匹配**：优雅地处理不同情况

rust

```rust
match divide(10, 2) {
    Ok(result) => println!("结果: {}", result),
    Err(e) => println!("错误: {}", e),
}
```

## 为什么这很重要？

传统语言（如早期 Java、C++）缺乏原生的 sum types，导致：

- 大量使用 `null`，引发空指针异常
- 错误处理不一致（异常、错误码混用）
- 类型系统无法表达"要么这样，要么那样"的语义

现代语言通过 sum types 解决了这些问题，让代码更安全、更清晰！