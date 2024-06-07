# 課題 4 のサンプルコード

> [!WARNING]
> これ以降は解答となる問題に対するサンプルコードと解説です。最初はこれを見ずに自分なりに頑張ってみてください。

## メインプログラム

```rust
fn main() {
    let mut words = vec![
        String::from("apple"),
        String::from("banana"),
        String::from("cherry"),
        String::from("date"),
        String::from("elderberry"),
    ];

    // ソート基準: 文字列の長さの降順
    words.sort_by(|a, b| b.len().cmp(&a.len()));

    println!("Sorted words by length (desc): {:?}", words);
}
```

## 解説

1. **文字列のベクタの定義**:

- `let mut words = vec![String::from("apple"), ...];`で文字列のベクタを作成します。

2. **カスタム比較クロージャ**:

- `words.sort_by(|a, b| b.len().cmp(&a.len()));`で文字列の長さの降順でソートします。
- `sort_by`メソッドはクロージャを受け取り、`cmp`メソッドを使用して長さを比較します。
- `b.len().cmp(&a.len())`は、`b`の長さを`a`の長さと比較し、降順にソートします。
