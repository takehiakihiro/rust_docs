# 課題 2

## 課題内容

1. コマンドライン引数で指定された複数の URL から HTML をダウンロードする。URL のリストは第二引数以降で指定します。
2. ダウンロードした HTML を、指定されたプリフィックスを使ってファイルに保存する。
3. ファイル名は`<prefix>_<number>.html` という形式にする。`prefix`は第一引数で指定します。`number`は指定された URL の順番で採番し、0 から始めます。
4. ダウンロードやファイル操作中に発生するエラーを thiserror を使って処理する。

### サンプルの Error や関数一覧

```rust
use std::env;
use std::fs::File;
use std::io::{self, Write};
use std::path::PathBuf;
use thiserror::Error;
use tokio::task;
use reqwest::Error as ReqwestError;

#[derive(Error, Debug)]
pub enum DownloadError {
    #[error("I/O Error")]
    Io(#[from] io::Error),
    #[error("Request Error")]
    Reqwest(#[from] ReqwestError),
}

async fn download_html(url: &str) -> Result<String, DownloadError> {
    ...
}

async fn save_html(prefix: &str, number: usize, html: &str) -> Result<(), DownloadError> {
    ...
}

async fn process_url(url: &str, prefix: &str, number: usize) -> Result<(), DownloadError> {
    ...
}

#[tokio::main]
async fn main() -> Result<(), DownloadError> {
    ...
}
```
