
这张幻灯片用表格总结了 Rust **Ownership & Borrowing（所有权与借用）** 的核心规则。

## 表格解读

|Type|Requirements|Access|
|---|---|---|
|**T**|Exactly one owner|Read & Write|
|**&T**|Only shared borrows can coexist|Read only|
|**&mut T**|No other borrows can coexist|Read & Write|