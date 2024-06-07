# 課題 6 のサンプルコード

> [!WARNING]
> これ以降は解答となる問題に対するサンプルコードと解説です。最初はこれを見ずに自分なりに頑張ってみてください。

## メインプログラム

```rust
use tokio::time::{sleep, Duration};
use tokio::task;

#[tokio::main]
async fn main() {
    let durations = vec![Duration::from_secs(1), Duration::from_secs(2), Duration::from_secs(3)];

    let tasks: Vec<_> = durations.into_iter().enumerate().map(|(i, duration)| {
        let task_id = i + 1;
        task::spawn(async move {
            println!("Task {}: Processing...", task_id);
            sleep(duration).await;
            println!("Task {}: Done", task_id);
        })
    }).collect();

    for task in tasks {
        task.await.unwrap();
    }
}
```

## 解説

1. **依存関係のインポート**:

   - `use tokio::time::{sleep, Duration};`で非同期のスリープ関数と時間型をインポートします。
   - `use tokio::task;`で非同期タスクを生成するための関数をインポートします。

2. **非同期タスクの生成**:

   - `let durations = vec![Duration::from_secs(1), Duration::from_secs(2), Duration::from_secs(3)];`で、各タスクのスリープ時間を保持するベクタを作成します。
   - `durations.into_iter().enumerate().map(|(i, duration)| { ... }).collect();`で、各スリープ時間に対してタスク ID を生成し、非同期タスクをクロージャで定義して生成します。

3. **クロージャ内での非同期処理**:

   - `task::spawn(async move { ... })`の中にクロージャを定義し、`task_id`と`duration`を使用してタスクの処理を行います。
   - `println!("Task {}: Processing...", task_id);`でタスクの開始をログに出力し、`sleep(duration).await;`で指定された時間だけ非同期スリープを行います。
   - `println!("Task {}: Done", task_id);`でタスクの終了をログに出力します。

4. **タスクの待機**:
   - `for task in tasks { task.await.unwrap(); }`で、すべてのタスクが完了するまで待機します。

この方法で、クロージャを使って非同期タスクを定義し、タスク内での処理を簡潔に記述しています。クロージャが活用される場面とその利便性を理解するのにふさわしいコード例となります。

## コラム : `into_iter()` と `enumerate()`

`into_iter().enumerate()`は Rust でよく使われるイディオムであり、特にベクタや配列を処理する際に便利です。この部分を詳しく解説します。

### `into_iter().enumerate()`の解説

#### `into_iter()`

- **`into_iter()`メソッド**は、所有権を持つイテレータを返します。これにより、ベクタの要素を消費し、各要素の所有権をイテレータが取得します。
- このメソッドは、`Vec<T>`や`[T; N]`のようなコレクションに対して呼び出されます。

#### `enumerate()`

- **`enumerate()`メソッド**は、イテレータをラップして、各要素にそのインデックスを付加します。これにより、元の要素とそのインデックスのペアを生成するイテレータが得られます。
- インデックスは`0`から始まり、順に増加します。

### `into_iter().enumerate()`の具体例

#### 例：`Vec`のイテレーションとインデックス付け

```rust
fn main() {
    let durations = vec![Duration::from_secs(1), Duration::from_secs(2), Duration::from_secs(3)];

    for (i, duration) in durations.into_iter().enumerate() {
        println!("Index: {}, Duration: {:?}", i, duration);
    }
}
```

このコードは以下のように動作します：

1. **`durations.into_iter()`**:

   - `durations`ベクタの要素を所有権ごとイテレータとして返します。
   - これにより、元のベクタは使えなくなります（要素の所有権がイテレータに移動するため）。

2. **`enumerate()`**:

   - 各要素にインデックスを付加したペアを生成します。
   - インデックスは`0`から始まり、各要素と一緒にタプル`(index, element)`として返されます。

3. **`for (i, duration) in ...`**:
   - イテレータから要素とそのインデックスを取り出し、ループ内で使用します。
   - ここで`i`はインデックス、`duration`は元の要素になります。

### まとめ

このイディオムを理解することで、コレクションの要素にインデックスを付加して処理する操作が簡潔に書けるようになります。特に、クロージャや非同期タスクと組み合わせることで、より柔軟で読みやすいコードを書くことができます。
