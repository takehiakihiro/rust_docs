# 課題 3 のサンプルコード

> [!WARNING]
> これ以降は解答となる問題に対するサンプルコードと解説です。最初はこれを見ずに自分なりに頑張ってみてください。

## メインプログラム

```rust
fn main() {
    let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];

    // フィルタリング条件: 偶数だけを抽出
    let filter_even = |x: &i32| -> bool { x % 2 == 0 };

    // マッピング処理: 2倍にする
    let map_double = |x: i32| -> i32 { x * 2 };

    let filtered_and_mapped: Vec<i32> = numbers
        .iter()
        .filter(filter_even)
        .map(|&x| map_double(x))
        .collect();

    println!("Filtered and mapped list: {:?}", filtered_and_mapped);
}
```

## 解説

1. **リストの定義**:

- `let numbers = vec![1, 2, 3, 4, 5, 6, 7, 8, 9, 10];`で整数のベクタを作成します。

2. **クロージャの定義**:

- `let filter_even = |x: &i32| -> bool { x % 2 == 0 };`で偶数をフィルタリングするクロージャを定義します。このクロージャは`x`が偶数かどうかをチェックし、結果を`bool`として返します。
- `let map_double = |x: i32| -> i32 { x * 2 };`で数値を 2 倍にするクロージャを定義します。このクロージャは`x`を 2 倍にして返します。

3. **フィルタリングとマッピング**:

- `numbers.iter().filter(filter_even).map(|&x| map_double(x)).collect();`で、リストをイテレートし、フィルタリングとマッピングを行います。
- `iter()`はイテレータを作成します。
- `filter(filter_even)`はクロージャを使って偶数のみをフィルタリングします。
- `map(|&x| map_double(x))`はクロージャを使って各要素を 2 倍にします。
- `collect()`は結果を`Vec<i32>`に収集します。
