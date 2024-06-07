# 課題 2 のサンプルコード

> [!WARNING]
> これ以降は解答となる問題に対するサンプルコードと解説です。最初はこれを見ずに自分なりに頑張ってみてください。

### Cargo.toml

まず、必要な依存関係を追加します。

```toml
[dependencies]
reqwest = { version = "0.12", features = ["json"] }
thiserror = "1.0"
tokio = { version = "1", features = ["full"] }
```

### メインプログラム

```rust
use reqwest::Error as ReqwestError;
use std::env;
use std::fs::File;
use std::io::{self, Write};
use thiserror::Error;
use tokio::task;

#[derive(Error, Debug)]
pub enum DownloadError {
    #[error("I/O Error")]
    Io(#[from] io::Error),
    #[error("Request Error")]
    Reqwest(#[from] ReqwestError),
}

async fn download_html(url: &str) -> Result<String, DownloadError> {
    let response = reqwest::get(url).await?;
    let html = response.text().await?;
    Ok(html)
}

async fn save_html(prefix: &str, number: usize, html: &str) -> Result<(), DownloadError> {
    let filename = format!("{}_{}.html", prefix, number);
    let mut file = File::create(filename)?;
    file.write_all(html.as_bytes())?;
    Ok(())
}

async fn process_url(url: &str, prefix: &str, number: usize) -> Result<(), DownloadError> {
    let html = download_html(url).await?;
    save_html(prefix, number, &html).await?;
    Ok(())
}

#[tokio::main]
async fn main() -> Result<(), DownloadError> {
    let args: Vec<String> = env::args().collect();
    if args.len() < 3 {
        eprintln!("Usage: {} <prefix> <url1> <url2> ...", args[0]);
        std::process::exit(1);
    }

    let prefix = &args[1];
    let urls = &args[2..];

    let mut tasks = Vec::new();

    for (i, url) in urls.iter().enumerate() {
        let prefix = prefix.clone();
        let url = url.clone();
        tasks.push(task::spawn(async move {
            match process_url(&url, &prefix, i).await {
                Ok(_) => println!("Successfully downloaded and saved: {}", url),
                Err(e) => eprintln!("Failed to process {}: {:?}", url, e),
            }
        }));
    }

    for task in tasks {
        let _ = task.await;
    }

    Ok(())
}
```

## 解説

### Cargo.toml

- thiserror: カスタムエラー型を簡単に定義するためのクレートです。
- tokio: 非同期プログラミングをサポートするためのクレートです。
- reqwest: HTTP クライアントとして使用するクレートです。

### カスタムエラー型の定義

```rust
#[derive(Error, Debug)]
pub enum DownloadError {
    #[error("I/O Error")]
    Io(#[from] io::Error),
    #[error("Request Error")]
    Reqwest(#[from] ReqwestError),
}
```

- `Io`: 標準の I/O エラーをラップします。`?`演算子を使用することで、自動的に`io::Error`を`DownloadError::Io`に変換します。
- `Reqwest`: `reqwest`のエラーをラップします。`?`演算子を使用することで、自動的に`reqwest::Error`を`DownloadError::Reqwest`に変換します。

### HTML をダウンロードする関数

```rust
async fn download_html(url: &str) -> Result<String, DownloadError> {
    let response = reqwest::get(url).await?;
    let html = response.text().await?;
    Ok(html)
}
```

- `download_html`関数: 指定された URL から HTML をダウンロードします。?演算子を使ってエラーを簡潔にハンドリングしています。

### HTML を保存する関数

```rust
async fn save_html(prefix: &str, number: usize, html: &str) -> Result<(), DownloadError> {
    let filename = format!("{}_{}.html", prefix, number);
    let mut file = File::create(filename)?;
    file.write_all(html.as_bytes())?;
    Ok(())
}
```

- save_html 関数: ダウンロードした HTML を指定されたファイル名で保存します。

### URL を処理する関数

```rust
async fn process_url(url: &str, prefix: &str, number: usize) -> Result<(), DownloadError> {
    let html = download_html(url).await?;
    save_html(prefix, number, &html).await?;
    Ok(())
}
```

- `process_url`関数: 指定された URL から HTML をダウンロードし、ファイルに保存します。エラーハンドリングを行い、適切なメッセージを出力します。

### メイン関数

```rust
#[tokio::main]
async fn main() -> Result<(), DownloadError> {
    let args: Vec<String> = env::args().collect();
    if args.len() < 3 {
        eprintln!("Usage: {} <prefix> <url1> <url2> ...", args[0]);
        std::process::exit(1);
    }

    let prefix = &args[1];
    let urls = &args[2..];

    let mut tasks = Vec::new();

    for (i, url) in urls.iter().enumerate() {
        let prefix = prefix.clone();
        let url = url.clone();
        tasks.push(task::spawn(async move {
            match process_url(&url, &prefix, i).await {
                Ok(_) => println!("Successfully downloaded and saved: {}", url),
                Err(e) => eprintln!("Failed to process {}: {:?}", url, e),
            }
        }));
    }

    for task in tasks {
        let _ = task.await;
    }

    Ok(())
}
```

- コマンドライン引数の処理: プログラムの引数を取得し、prefix と urls として保存します。
- 非同期タスクの生成: 各 URL に対して非同期タスクを生成し、並列に処理します。
- タスクの完了を待つ: すべてのタスクが完了するまで待機します。

## まとめ

このプログラムは、以下のポイントを学ぶことができます。

1. `thiserror` を使ったカスタムエラー型の定義とエラーハンドリング。
2. `?`演算子を使った簡潔なエラーハンドリング。
3. 非同期プログラミングの基本と並列処理。
4. `reqwest` を使った HTTP リクエストの処理とエラーハンドリング。
5. ファイル操作の基本。

この課題を通じて、Rust のエラーハンドリングや非同期プログラミングの実用的な技術を学ぶことができます。
