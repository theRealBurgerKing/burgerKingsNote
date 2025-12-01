
这张幻灯片用表格总结了 Rust **Ownership & Borrowing（所有权与借用）** 的核心规则。

## 表格解读

| Type       | Requirements                    | Access       |
| ---------- | ------------------------------- | ------------ |
| **T**      | Exactly one owner               | Read & Write |
| **&T**     | Only shared borrows can coexist | Read only    |
| **&mut T** | No other borrows can coexist    | Read & Write |





结合 `slice.rs` 讲解 **Slices（切片）** 的核心价值。

## 问题：代码重复且不灵活

rust

```rust
fn sum_array(list: [i32; 10]) -> i32 {  // ❌ 只能处理长度为 10 的数组
    let mut sum = 0;
    for elem in list {
        sum += elem;
    }
    sum
}

fn sum_vec(list: Vec<i32>) -> i32 {  // ❌ 只能处理 Vec
    let mut sum = 0;
    for elem in list {
        sum += elem;
    }
    sum
}
```

**问题：**

- 两个函数逻辑完全相同
- 无法处理数组的部分元素
- 无法处理其他长度的数组
- `sum_vec` 获取所有权，调用后原 Vec 失效

## 解决方案：使用 Slice

rust

```rust
fn sum_slice(list: &[i32]) -> i32 {  // ✅ 统一接口
    let mut sum = 0;
    for elem in list {
        sum += elem;
    }
    sum
}
```

## Slice 的强大之处

**1. 统一所有连续数据类型**

rust

```rust
let arr = [1, 2, 3, 4, 5];
sum_slice(&arr);              // ✅ 数组

let vec = vec![1, 2, 3];
sum_slice(&vec);              // ✅ Vec

let boxed = Box::new([1, 2]); 
sum_slice(&boxed);            // ✅ Box<[T]>
```

**2. 可以引用部分数据**

rust

```rust
let arr = [1, 2, 3, 4, 5];
sum_slice(&arr[1..4]);        // ✅ [2, 3, 4]
sum_slice(&arr[..2]);         // ✅ [1, 2]
sum_slice(&arr[3..]);         // ✅ [4, 5]
```

**3. 不获取所有权**

rust

````rust
let vec = vec![1, 2, 3];
let total = sum_slice(&vec);  // 借用
println!("{:?}", vec);        // ✅ vec 仍然可用
```

## Slice 的内部结构
```
[ptr: *const T, len: usize]
  ↓
  指向实际数据
````