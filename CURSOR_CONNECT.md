# Cursor から Strata に繋ぐ

Strata が起動していること（`http://127.0.0.1:8080/health` が `loaded: true`）。

## OpenAI 互換プロバイダ

| 項目 | 値 |
|------|-----|
| Base URL | `http://127.0.0.1:8080/v1` |
| API Key | 任意の文字列（未設定なら何でも可） |
| Model | 任意（例: `strata` / `coder`） |

Cursor Settings → Models → カスタム OpenAI 互換エンドポイントがあれば上記を設定。

## Anthropic 互換

- Base: `http://127.0.0.1:8080`
- Messages: `http://127.0.0.1:8080/v1/messages`

## MCP（起動・停止・ログ）

ユーザー／プロジェクトの `mcp.json` に `strata` サーバを追加済み。

Cursor 再起動後、ツール `strata_status` / `strata_start` / `strata_stop` / `strata_logs` が使える。

## llama.cpp との使い分け

同時に両方の重いモデルを載せない。Strata 使用時は `D:\work\llama_cpp` のサーバを止める。
