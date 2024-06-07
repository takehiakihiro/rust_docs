# 課題 5 のサンプルコード

> [!WARNING]
> これ以降は解答となる問題に対するサンプルコードと解説です。最初はこれを見ずに自分なりに頑張ってみてください。

## メインプログラム

```rust
fn main() {
    let operations = vec![
        |x: i32| -> Result<i32, &'static str> { if x > 0 { Ok(x + 1) } else { Err("Negative input") } },
        |x: i32| -> Result<i32, &'static str> { if x % 2 == 0 { Ok(x * 2) } else { Err("Not even") } },
        |x: i32| -> Result<i32, &'static str> { if x < 10 { Ok(x - 3) } else { Err("Too large") } },
    ];

    let input = 4;
    let mut result = Ok(input);

    for operation in operations {
        result = result.and_then(operation);
        if let Err(e) = result {
            println!("Operation failed: {}", e);
            break;
        }
    }

    match result {
        Ok(value) => println!("Final result: {}", value),
        Err(_) => println!("Processing failed."),
    }
}
```

## 解説

1. **操作のベクタの定義**:

   - `let operations = vec![...]`で、`Result`を返す 3 つのクロージャのベクタを作成します。

2. **初期入力**:

   - `let input = 4;`で初期値を設定します。
   - `let mut result = Ok(input);`で、初期値を`Result`型にラップします。

3. **操作の実行**:

   - `for operation in operations { ... }`で、各操作を順次実行します。
   - `result = result.and_then(operation);`で、現在の`result`が`Ok`であれば次の操作を実行し、`Err`であればエラーを伝播します。
   - `if let Err(e) = result { ... }`で、エラーが発生した場合にエラーメッセージを表示し、ループを抜けます。

4. **最終結果の表示**:
   - `match result { ... }`で最終結果を表示します。`Ok`であれば最終結果を表示し、`Err`であればエラーメッセージを表示します。
