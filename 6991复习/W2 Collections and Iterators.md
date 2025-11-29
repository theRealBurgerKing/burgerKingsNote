## Rust 常见集合总览

### 1. **Array（数组）**

**固定大小**的同类型元素集合，存储在**栈**上。

rust

````rust
// 声明方式
let arr1: [i32; 5] = [1, 2, 3, 4, 5];  // 显式指定类型和大小
let arr2 = [0; 10];  // 创建10个0
let arr3 = [1, 2, 3];  // 类型推断

// 访问元素
println!("{}", arr1[0]);  // 1
println!("{}", arr1.len());  // 5

// 遍历
for item in &arr1 {
    println!("{}", item);
}

// 特点
// ✓ 大小固定，编译时确定
// ✓ 存储在栈上，性能高
// ✗ 不能增长或缩小
```

**内存布局：**
```
栈
+------------------+
| [1, 2, 3, 4, 5] |  ← 连续存储
+------------------+
````

---

### 2. **Vec（向量）**

**可变大小**的同类型元素集合，存储在**堆**上。

rust

````rust
// 创建方式
let mut vec1 = Vec::new();
let mut vec2 = vec![1, 2, 3];  // 使用宏
let vec3: Vec<i32> = Vec::with_capacity(10);  // 预分配容量

// 操作
vec2.push(4);        // 添加元素
vec2.pop();          // 移除末尾元素
vec2.insert(0, 0);   // 在索引0处插入
vec2.remove(1);      // 移除索引1的元素

// 访问
println!("{}", vec2[0]);
println!("{:?}", vec2.get(10));  // 安全访问，返回 Option

// 遍历
for item in &vec2 {
    println!("{}", item);
}

// 特点
// ✓ 动态大小，可增长
// ✓ O(1) 追加（平摊）
// ✗ 插入/删除中间元素 O(n)
```

**内存布局：**
```
栈                    堆
+----------+       +------------------+
| ptr      |-----> | 1, 2, 3, 4      |
| len: 4   |       | (capacity: 8)   |
| cap: 8   |       +------------------+
+----------+
````

---

### 3. **VecDeque（双端队列）**

可以从**两端**高效添加/移除元素的队列。

rust

```rust
use std::collections::VecDeque;

let mut deque = VecDeque::new();

// 从两端操作
deque.push_back(1);   // 尾部添加
deque.push_front(0);  // 头部添加
deque.pop_back();     // 尾部移除
deque.pop_front();    // 头部移除

// 使用场景
// ✓ 需要从两端操作
// ✓ 实现队列或栈
// ✓ 滑动窗口算法

// 示例：队列
let mut queue = VecDeque::new();
queue.push_back(1);
queue.push_back(2);
let first = queue.pop_front();  // Some(1)
```

---

### 4. **String（字符串）**

可变、可增长的 **UTF-8** 编码字符串。

rust

````rust
// 创建
let mut s1 = String::new();
let s2 = String::from("hello");
let s3 = "world".to_string();

// 操作
s1.push_str("hello");  // 添加字符串
s1.push('!');          // 添加单个字符
let s4 = s2 + " " + &s3;  // 连接（s2 被移动）
let s5 = format!("{} {}", "hello", "world");  // 格式化

// 访问（注意：不能直接索引！）
// let c = s2[0];  // ❌ 错误！
let bytes = s2.as_bytes();  // 字节数组
let chars: Vec<char> = s2.chars().collect();  // 字符数组

// 特点
// ✓ UTF-8 编码，支持多语言
// ✗ 不能通过索引直接访问字符（因为 UTF-8 是变长编码）
```

**内存布局：**
```
栈                    堆
+----------+       +------------------+
| ptr      |-----> | h e l l o (UTF-8)|
| len: 5   |       |                  |
| cap: 8   |       +------------------+
+----------+
````

---

### 5. **LinkedList（链表）**

双向链表，每个元素分散存储。

rust

```rust
use std::collections::LinkedList;

let mut list = LinkedList::new();

list.push_back(1);
list.push_front(0);
list.pop_back();
list.pop_front();

// 使用场景（较少使用）
// ✓ 需要频繁在中间插入/删除
// ✗ 访问元素慢 O(n)
// ✗ 缓存局部性差

// 注意：在 Rust 中，Vec 通常比 LinkedList 更快！
```

---

### 6. **HashMap（哈希映射）**

键值对集合，基于哈希表实现。

rust

```rust
use std::collections::HashMap;

// 创建
let mut map = HashMap::new();

// 插入
map.insert("key1", 10);
map.insert("key2", 20);

// 访问
let value = map.get("key1");  // Some(&10)
let value2 = map.get("key3");  // None

// 更新
map.insert("key1", 15);  // 覆盖
map.entry("key3").or_insert(30);  // 不存在则插入

// 遍历
for (key, value) in &map {
    println!("{}: {}", key, value);
}

// 特点
// ✓ O(1) 平均查找/插入
// ✗ 无序
// ✗ 键必须实现 Hash + Eq
```

**示例：单词计数**

rust

```rust
let text = "hello world hello rust";
let mut word_count = HashMap::new();

for word in text.split_whitespace() {
    let count = word_count.entry(word).or_insert(0);
    *count += 1;
}

println!("{:?}", word_count);
// {"hello": 2, "world": 1, "rust": 1}
```

---

### 7. **BTreeMap（B树映射）**

**有序**的键值对集合，基于 B 树实现。

rust

```rust
use std::collections::BTreeMap;

let mut map = BTreeMap::new();
map.insert(3, "three");
map.insert(1, "one");
map.insert(2, "two");

// 遍历是有序的
for (key, value) in &map {
    println!("{}: {}", key, value);
}
// 输出: 1: one, 2: two, 3: three

// 范围查询
for (key, value) in map.range(1..=2) {
    println!("{}: {}", key, value);
}

// 特点
// ✓ 键有序
// ✓ O(log n) 查找/插入
// ✓ 支持范围查询
// ✗ 比 HashMap 稍慢
```

---

### 8. **HashSet（哈希集合）**

**无序**、**不重复**元素的集合。

rust

```rust
use std::collections::HashSet;

let mut set = HashSet::new();

// 插入
set.insert(1);
set.insert(2);
set.insert(1);  // 重复，不会添加

// 检查存在
println!("{}", set.contains(&1));  // true

// 集合操作
let set1: HashSet<_> = [1, 2, 3].iter().cloned().collect();
let set2: HashSet<_> = [2, 3, 4].iter().cloned().collect();

let union: HashSet<_> = set1.union(&set2).cloned().collect();
let intersection: HashSet<_> = set1.intersection(&set2).cloned().collect();
let difference: HashSet<_> = set1.difference(&set2).cloned().collect();

println!("{:?}", union);         // {1, 2, 3, 4}
println!("{:?}", intersection);  // {2, 3}
println!("{:?}", difference);    // {1}

// 特点
// ✓ O(1) 查找/插入
// ✓ 自动去重
// ✗ 无序
```

---

### 9. **BTreeSet（B树集合）**

**有序**、**不重复**元素的集合。

rust

```rust
use std::collections::BTreeSet;

let mut set = BTreeSet::new();
set.insert(3);
set.insert(1);
set.insert(2);

// 遍历是有序的
for item in &set {
    println!("{}", item);
}
// 输出: 1, 2, 3

// 范围查询
for item in set.range(1..=2) {
    println!("{}", item);
}

// 特点
// ✓ 元素有序
// ✓ O(log n) 查找/插入
// ✓ 支持范围查询
```

---

### 10. **BinaryHeap（二叉堆）**

优先队列，最大堆实现。

rust

```rust
use std::collections::BinaryHeap;

let mut heap = BinaryHeap::new();

// 插入
heap.push(1);
heap.push(5);
heap.push(3);

// 获取最大值
println!("{:?}", heap.peek());  // Some(&5)

// 移除最大值
println!("{:?}", heap.pop());  // Some(5)
println!("{:?}", heap.pop());  // Some(3)
println!("{:?}", heap.pop());  // Some(1)

// 使用场景
// ✓ 实现优先队列
// ✓ 需要快速访问最大/最小元素
// ✓ Dijkstra 算法等

// 最小堆示例
use std::cmp::Reverse;
let mut min_heap = BinaryHeap::new();
min_heap.push(Reverse(5));
min_heap.push(Reverse(1));
min_heap.push(Reverse(3));
println!("{:?}", min_heap.pop());  // Some(Reverse(1))
```