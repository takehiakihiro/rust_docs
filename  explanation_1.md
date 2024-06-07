# 課題 1 のサンプルコード

> [!WARNING]
> これ以降は解答となる問題に対するサンプルコードと解説です。最初はこれを見ずに自分なりに頑張ってみてください。

## Cargo.toml

まず、必要な依存関係を追加します。

```toml
[dependencies]
tokio = { version = "1", features = ["full"] }
serde = { version = "1", features = ["derive"] }
serde_json = "1"
```

## サーバーの実装

```rust
use tokio::net::TcpListener;
use tokio::io::{AsyncReadExt, AsyncWriteExt};
use serde::{Serialize, Deserialize};
use std::env;

#[derive(Serialize, Deserialize)]
struct Request {
    header: String,
    body: String,
}

#[derive(Serialize, Deserialize)]
struct Response {
    header: String,
    body: String,
}

async fn handle_client(mut socket: tokio::net::TcpStream, response_header: String, append_str: String) {
    let mut buffer = [0; 1024];
    loop {
        let n = match socket.read(&mut buffer).await {
            Ok(n) if n == 0 => return,
            Ok(n) => n,
            Err(e) => {
                eprintln!("failed to read from socket; err = {:?}", e);
                return;
            }
        };

        let request: Request = match serde_json::from_slice(&buffer[..n]) {
            Ok(req) => req,
            Err(e) => {
                eprintln!("failed to parse JSON; err = {:?}", e);
                continue;
            }
        };

        let response = Response {
            header: response_header.clone(),
            body: format!("{}{}", request.body, append_str),
        };

        let response_json = match serde_json::to_string(&response) {
            Ok(json) => json,
            Err(e) => {
                eprintln!("failed to serialize JSON; err = {:?}", e);
                continue;
            }
        };

        if let Err(e) = socket.write_all(response_json.as_bytes()).await {
            eprintln!("failed to write to socket; err = {:?}", e);
            return;
        }
    }
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let args: Vec<String> = env::args().collect();
    if args.len() != 3 {
        eprintln!("Usage: {} <response_header> <append_string>", args[0]);
        std::process::exit(1);
    }
    let response_header = args[1].clone();
    let append_str = args[2].clone();

    let listener = TcpListener::bind("127.0.0.1:8080").await?;
    println!("Server running on port 8080");

    loop {
        let (socket, _) = listener.accept().await?;
        let response_header = response_header.clone();
        let append_str = append_str.clone();
        tokio::spawn(async move {
            handle_client(socket, response_header, append_str).await;
        });
    }
}
```

## クライアントの実装

```rust
use tokio::net::TcpStream;
use tokio::io::{self, AsyncWriteExt, AsyncReadExt};
use serde::{Serialize, Deserialize};
use std::env;
use std::io::{stdin, BufRead};

#[derive(Serialize, Deserialize)]
struct Request {
    header: String,
    body: String,
}

#[derive(Serialize, Deserialize)]
struct Response {
    header: String,
    body: String,
}

#[tokio::main]
async fn main() -> Result<(), Box<dyn std::error::Error>> {
    let args: Vec<String> = env::args().collect();
    if args.len() != 2 {
        eprintln!("Usage: {} <request_header>", args[0]);
        std::process::exit(1);
    }
    let request_header = args[1].clone();

    let mut socket = TcpStream::connect("127.0.0.1:8080").await?;
    println!("Connected to the server.");

    let stdin = io::stdin();
    let mut handle = stdin.lock();

    loop {
        let mut input = String::new();
        handle.read_line(&mut input)?;

        let request = Request {
            header: request_header.clone(),
            body: input.trim().to_string(),
        };

        let request_json = serde_json::to_string(&request)?;

        socket.write_all(request_json.as_bytes()).await?;

        let mut buffer = [0; 1024];
        let n = socket.read(&mut buffer).await?;

        let response: Response = serde_json::from_slice(&buffer[..n])?;
        println!("Response from server: {:?}", response);
    }
}
```

# 解説

## サーバーの実装

1. コマンドライン引数の処理：`env::args()`を使ってプログラムの引数を取得し、`response_header`と`append_str`として保存します。
2. `handle_client`関数に`response_header`と`append_str`を渡す：クライアント接続ごとにこれらの値を渡して、レスポンスに含めます。
3. 同時接続の処理：`tokio::spawn`を使って、新しいクライアント接続ごとに非同期タスクを生成し、複数のクライアントとの独立したやり取りを実現します。

## クライアントの実装

1. コマンドライン引数の処理：`env::args()`を使ってプログラムの引数を取得し、`request_header`として保存します。
2. `Request`構造体に`request_header`を設定：標準入力から取得したデータと一緒に`request_header`をサーバーに送信します。
3. レスポンスの表示：サーバーから受け取ったレスポンスの JSON をそのまま標準出力に表示します。

これにより、サーバーとクライアントのヘッダ文字列をプログラムの起動時に引数で渡すことができ、指定されたヘッダを使って通信を行うことができます。また、サーバーはリクエストの body に追加する文字列も引数から受け取り、レスポンスに含めます。さらに、サーバーは同時に複数のクライアントとやり取りができ、それぞれ独立してやり取りができるようになります。
