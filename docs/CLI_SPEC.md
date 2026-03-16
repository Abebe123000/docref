# docref コマンド仕様

## コマンド一覧

| コマンド | 説明 |
|----------|------|
| `list` | パーマネントリンクを一覧表示 |
| `check` | 古いリンクを検出 |
| `update` | リンクを最新化 |
| `sync` | AI による実装の同期 |
| `init` | プロジェクトの初期化 |

---

## list - リンクの一覧表示

指定されたファイル内の仕様書へのパーマネントリンクを検出し、一覧表示します。

```bash
docref list [PATH]
```

**オプション:**
| オプション | 説明 |
|------------|------|
| `--format <FORMAT>` | 出力形式 (`text`, `json`, `markdown`) デフォルト: `text` |
| `--recursive, -r` | ディレクトリを再帰的にスキャン |
| `--filter <PATTERN>` | 特定の仕様ファイルを参照するリンクのみ表示 |

**出力例:**
```
Found 3 specification links:

src/api.rs:12
  https://github.com/owner/repo/blob/abc123/docs/api-spec.md#L15-L30
  Status: current (matches latest on main)

src/handler.rs:45
  https://github.com/owner/repo/blob/abc123/docs/api-spec.md#L50-L65
  Status: outdated (spec modified in 3 commits)

src/model.rs:8
  https://github.com/owner/repo/blob/def456/docs/data-model.md#L1-L20
  Status: stale (file deleted)
```

---

## check - 古いリンクの検出

パーマネントリンクが指す仕様書と、現在の最新の仕様書を比較し、乖離を検出します。

```bash
docref check [PATH]
```

**オプション:**
| オプション | 説明 |
|------------|------|
| `--branch <BRANCH>` | 比較対象のブランチ（省略時は自動検出） |
| `--strict` | 1行でも変更があればエラーとする |
| `--exit-code` | 古いリンクがある場合に非ゼロで終了（CI用） |
| `--show-diff` | 仕様書の変更内容を表示 |

**比較対象ブランチの決定（優先順位）:**
1. `--branch` オプションで明示的に指定された場合はそれを使用
2. `.docref.toml` の `default_branch` が設定されている場合はそれを使用
3. Git コマンドでリポジトリのデフォルトブランチを取得

**出力例:**
```
Checking 3 specification links...

✗ src/handler.rs:45
  Link: https://github.com/owner/repo/blob/abc123/docs/api-spec.md#L50-L65
  Issue: Specification changed (5 lines modified)
  Last modified: 2024-01-15 by @username (commit: def789)

  Diff:
  - Rate limit: 100 requests per minute
  + Rate limit: 50 requests per minute

✗ src/model.rs:8
  Link: https://github.com/owner/repo/blob/def456/docs/data-model.md#L1-L20
  Issue: Specification file deleted in commit ghi012

Summary: 2 outdated specification links found
```

---

## update - リンクの最新化

古いパーマネントリンクを最新のコミット SHA に更新します。

```bash
docref update [PATH]
```

**オプション:**
| オプション | 説明 |
|------------|------|
| `--dry-run` | 実際には更新せず、変更内容をプレビュー |
| `--branch <BRANCH>` | 更新先のブランチ（省略時は自動検出） |
| `--interactive, -i` | 各リンクについて更新するか確認 |
| `--preserve-lines` | 行番号を維持（仕様書内の移動を追跡しない） |

**出力例:**
```
Updating specification links...

src/handler.rs:45
  - https://github.com/owner/repo/blob/abc123/docs/api-spec.md#L50-L65
  + https://github.com/owner/repo/blob/xyz789/docs/api-spec.md#L55-L70
  Lines shifted: +5

Updated 1 link in 1 file.

⚠ Warning: The specification content has changed. Please review if the
  implementation in src/handler.rs:45-80 still matches the updated spec.
```

---

## sync - AI による実装の同期

仕様書の変更に基づき、AI を使用してコードの実装を更新提案します。

```bash
docref sync [PATH]
```

**オプション:**
| オプション | 説明 |
|------------|------|
| `--dry-run` | 実際には更新せず、提案内容をプレビュー |
| `--model <MODEL>` | 使用する AI モデル |
| `--interactive, -i` | 各変更について適用するか確認 |
| `--update-links` | 同期後にリンクも最新化 |

**出力例:**
```
Analyzing specification changes...

src/handler.rs:45-80
  Specification change detected:
  - Rate limit: 100 requests per minute
  + Rate limit: 50 requests per minute

  Current implementation:
    const RATE_LIMIT: u32 = 100;

  Suggested update:
    const RATE_LIMIT: u32 = 50;

  Apply this change? [y/N]
```

---

## init - プロジェクトの初期化

設定ファイルを生成し、プロジェクトをセットアップします。

```bash
docref init
```

**動作:**
1. `.docref.toml` を生成
2. Git リモートからリポジトリ情報を自動検出
3. 既存のパーマネントリンクをスキャンしてレポート

---

## リンクの検出方式

ファイル内の Git パーマネントリンク URL を正規表現でマッチします。コメント形式や言語を問わず、全てのテキストファイルから検出します。

**検出例:**

```rust
// ソースコードのコメント
// See: https://github.com/owner/repo/blob/abc123/docs/spec.md#L10-L20
fn validate_request() { ... }
```

```markdown
<!-- Markdown ドキュメント -->
この機能は [仕様](https://github.com/owner/repo/blob/abc123/docs/spec.md#L10-L20) に基づいています。
```

```yaml
# テスト仕様書
test_case:
  description: "認証フローのテスト"
  spec_ref: https://github.com/owner/repo/blob/abc123/docs/auth-spec.md#L5-L20
```

**対応ファイル:**
- ソースコード（`.rs`, `.ts`, `.py`, `.go` など）
- ドキュメント（`.md`, `.txt`, `.adoc` など）
- 設定ファイル（`.yaml`, `.toml`, `.json` など）
- その他全てのテキストファイル

---

## パーマネントリンクの検出パターン

以下の URL パターンを検出します：

```
# 基本形式
https://github.com/{owner}/{repo}/blob/{commit_sha}/{path}

# 行指定あり
https://github.com/{owner}/{repo}/blob/{commit_sha}/{path}#L{line}

# 範囲指定あり
https://github.com/{owner}/{repo}/blob/{commit_sha}/{path}#L{start}-L{end}
```

**commit SHA の識別:**
- 40文字の完全な SHA: `abc123def456...`
- 7文字以上の短縮 SHA: `abc123d`
- ブランチ名・タグ名は対象外（これらはパーマネントリンクではないため警告を出す）

---

## 設定ファイル

プロジェクトルートに `.docref.toml` を配置して設定をカスタマイズできます。

```toml
# .docref.toml

[general]
# スキャン対象のファイルパターン
include = ["src/**/*.rs", "lib/**/*.rs"]
# 除外パターン
exclude = ["target/**", "tests/**"]

[git]
# 比較対象のデフォルトブランチ（省略時は Git から自動検出）
# default_branch = "main"

[cache]
# 外部リポジトリのキャッシュディレクトリ
dir = "~/.cache/docref/repos"
# キャッシュの有効期限（自動 fetch の間隔）
ttl = "1h"
# コマンド実行時に自動で fetch するか
auto_fetch = true

[check]
# 許容する変更の閾値（行数）
tolerance_lines = 0
# 空白のみの変更を無視
ignore_whitespace = true

[sync]
# 使用する AI プロバイダー
ai_provider = "anthropic"  # or "openai", "local"
# モデル名
ai_model = "claude-sonnet-4-5-20250929"
```

---

## 終了コード

| コード | 意味 |
|--------|------|
| 0 | 正常終了 |
| 1 | 古いリンクが検出された（`--exit-code` 使用時） |
| 2 | エラー（ファイルが見つからない、Git エラーなど） |

---

## 対応プラットフォーム

Git ベースのため、以下のプラットフォームに対応しています：

- GitHub (`https://github.com/...`)
- GitLab (`https://gitlab.com/...`)
- Bitbucket (`https://bitbucket.org/...`)
- 自前 Git サーバー（URL パターンを設定で追加可能）
