# 課題 4

ソートとカスタム比較の定義の仕方を学ぶ課題です。

## 課題内容

1. 文字列のリストを長さの降順でソートしてください。
2. ソート基準をクロージャで定義してください。

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
    words.sort_by(/* ここを書く */);

    println!("Sorted words by length (desc): {:?}", words);
}
```

「ここを書く」をクロージャで一行で書いてみてください。
