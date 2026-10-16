# Strata — 本機セットアップ記録

**日付:** 2026-10-16  
**PC:** MouseComputer X570-PG4, RAM 64 GB  
**GPU:** AMD Radeon RX 5700 (VRAM 8 GB, gfx1010), Adrenalin 32.0.21045.5002  
**方針:** 実験オプトイン `STRATA_EXPERIMENTAL_GFX1010=1` + 自前 HIP ビルド

## 配置

| 項目 | パス |
|------|------|
| ソース | `D:\work\Strata` |
| モデル・pack | `D:\Strata-data` |
| モデル | Coder IQ1_M (`qwen3.8-flash-next-coder-iq1_m`) |
| 起動 | `run-coder-iq1_m.bat` または `START-HERE.bat` |
| 設定 | `strata-coder-iq1_m.json` |
| ログ | `strata-coder-iq1_m.log` |
| API | `http://127.0.0.1:8080/v1` |

## 本機向けパッチ / ビルド

1. `setup.py` に `STRATA_EXPERIMENTAL_GFX1010=1`（PCI `0x731F` → gfx1010）を追加
2. `tools\hip\build_windows.bat` を `STRATA_HIP_ARCHS=gfx1010` で実行 → `dist\strata-windows-x64-hip.zip`
3. Windows の `pathlib.replace` で `engine\rocm` 展開が失敗したため、zip は手動 `shutil.move` で展開

## チューニング（`strata-coder-iq1_m.json`）

- `context` 32768 / KV int8 / vision off / parallel 1
- `"draft_vocab": "en"`（8 GB カード向け draft 語彙）
- `"--adapt-async", "1"`
- `"env": { "STRATA_PIN_BY_NEED": "1" }`
- hipBLASLt チューニング表は gfx1010 に無し → plain hipBLAS

## 疎通・ベンチ（2026-10-16）

| 指標 | 値 |
|------|-----|
| `/health` | `loaded: true` |
| 短文 decode | ~11.1 tok/s（初回） |
| コーディングプロンプト decode | **14.2 tok/s**（148 tokens） |
| 同 prompt 読み込み | **20.0 tok/s**（43 tokens） |
| 詳細 | `LOCAL_BENCH.json` |

コミュニティ RX 5700 XT の Coder 報告（~18–23 tok/s）より低いが、非 XT 8 GB + Windows HIP 実験パスとしては妥当。

## 起動・停止

```powershell
# 起動（別ウィンドウ）
Start-Process .\run-coder-iq1_m.bat

# 疎通
Invoke-RestMethod http://127.0.0.1:8080/health

# 停止: ウィンドウを閉じる、または strata.exe / server.py を終了
```

環境変数が必要なチェック／再セットアップ:

```powershell
$env:STRATA_EXPERIMENTAL_GFX1010 = "1"
.\.venv\Scripts\python.exe setup.py --check --backend hip
```

`START-HERE.bat` 経由だと env が伝わらないことがある → 上記のように `python setup.py` を直接使う。

## Cursor / アプリ接続

- OpenAI 互換 Base URL: `http://127.0.0.1:8080/v1`
- API key: 任意（未設定）
- モデル名: 任意（実体は Coder IQ1_M）
- Anthropic: `http://127.0.0.1:8080/v1/messages`
- MCP: Cursor `mcp.json` に `tools/strata_mcp.py` を登録済み

## 注意

- llama.cpp（`D:\work\llama_cpp`）と同時に重いモデルを載せない（RAM 競合）
- 初回ロードは 1–3 分マシンが重くなる
- gfx1010 は実験パス；メンテナ未検証
