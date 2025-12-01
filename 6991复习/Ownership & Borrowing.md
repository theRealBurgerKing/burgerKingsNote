这张幻灯片介绍 Rust 中的核心概念：**Owned vs Copy**（所有权 vs 复制），以及相关的三个问题。这是理解 Rust 所有权（ownership）和借用（borrowing）系统的关键。

## 三个核心问题

### 1. What does it mean to be an "owned" type?

**什么是"拥有所有权"的类型？**

**Owned 类型**指的是拥有其数据所有权的类型。当变量拥有数据时：

rust

```rust
let s = String::from("hello");  // s 拥有这个字符串数据
```

**所有权规则：**

- 每个值在 Rust 中都有一个**所有者（owner）**
- 同一时间只能有**一个所有者**
- 当所有者离开作用域，值会被**自动释放（drop）**

**示例：所有权转移（Move）**

rust

```rust
let s1 = String::from("hello");
let s2 = s1;  // 所有权从 s1 转移到 s2
// println!("{}", s1);  // ❌ 错误！s1 不再有效
println!("{}", s2);     // ✅ 只有 s2 能访问数据
```

### 2. What types are Copy?

**哪些类型是 Copy 的？**

**Copy 类型**在赋值时会创建**值的副本**，而不是转移所有权。

**Copy 类型包括：**

- 所有整数类型：`i32`, `u64`, `usize` 等
- 浮点数：`f32`, `f64`
- 布尔值：`bool`
- 字符：`char`
- 元组（如果所有元素都是 Copy）：`(i32, i32)`
- 不可变引用：`&T`

**示例：Copy 类型的行为**

rust

```rust
let x = 5;
let y = x;  // 复制值，不是转移所有权
println!("{}, {}", x, y);  // ✅ x 和 y 都有效
```

**非 Copy 类型（Owned）：**

- `String`（堆分配的字符串）
- `Vec<T>`（动态数组）
- `Box<T>`（堆分配的指针）
- 自定义结构体（默认）

### 3. Can I make a new Copy type?

**我能创建新的 Copy 类型吗？**

**可以，但有条件！**

**为自定义类型实现 Copy：**

rust

```rust
#[derive(Copy, Clone)]
struct Point {
    x: i32,
    y: i32,
}

let p1 = Point { x: 1, y: 2 };
let p2 = p1;  // 复制，不是转移
println!("{}, {}", p1.x, p2.x);  // ✅ 都有效
```

**限制条件：** 只有当结构体的**所有字段都是 Copy** 时，才能派生 `Copy`：

rust

```rust
// ❌ 错误：不能为包含 String 的类型实现 Copy
#[derive(Copy, Clone)]
struct Person {
    name: String,  // String 不是 Copy
    age: i32,
}
```

rust

```rust
// ✅ 正确：所有字段都是 Copy
#[derive(Copy, Clone)]
struct Person {
    id: u32,      // u32 是 Copy
    age: i32,     // i32 是 Copy
}
```

## Owned vs Copy 对比

| 特性       | Owned 类型        | Copy 类型       |
| -------- | --------------- | ------------- |
| **赋值行为** | 转移所有权（Move）     | 复制值           |
| **旧变量**  | 赋值后失效           | 仍然有效          |
| **性能**   | 无额外开销（只转移指针）    | 可能有复制开销       |
| **典型例子** | `String`, `Vec` | `i32`, `bool` |
| **内存位置** | 通常在堆上           | 通常在栈上         |




















这张幻灯片介绍 **Cascading ownership（级联所有权）** 的概念。

## 什么是级联所有权？

**核心概念：** "If X owns Y and Y owns Z..."

- 如果 X 拥有 Y，Y 拥有 Z...
- 那么当 X 被释放时，Y 和 Z 也会**自动级联释放**

这是 Rust 所有权系统的一个重要特性：**所有权链条的传递性**。

## 示例：Student name

幻灯片提到的例子是"学生姓名"，让我们用代码说明：

rust

````rust
struct Student {
    name: String,        // Student 拥有 String
    id: u32,
}

fn main() {
    let student = Student {
        name: String::from("Alice"),  // String 拥有堆上的 "Alice" 数据
        id: 12345,
    };
    
    // 所有权链条：
    // student -> name (String) -> "Alice" (堆数据)
    
} // student 离开作用域
  // ↓ 自动释放 student
  // ↓ 级联释放 name (String)
  // ↓ 级联释放 "Alice" (堆数据)
```

## 级联所有权的工作原理

### 可视化所有权树：
```
student (栈上)
  └── name: String (栈上的智能指针)
       └── "Alice" (堆上的实际数据)
  └── id: u32 (栈上)
````

**当 `student` 离开作用域时：**

1. Rust 调用 `student` 的析构函数（`Drop`）
2. `name` 字段被释放，触发 `String` 的析构函数
3. `String` 释放堆上的 `"Alice"` 数据
4. `id` 字段被释放（Copy 类型，直接清空栈内存）