# docref ユーザーストーリー

## 概要

`docref` は、ソースコード内に埋め込まれた仕様書へのパーマネントリンクを管理するための CLI ツールです。

## 背景

コードを実装する際、「この実装はどの仕様に基づいているか」を明示することは重要です。Git のパーマネントリンクを使えば、仕様書の特定時点の内容を正確に参照できます。

```rust
// この関数は以下の仕様に基づいて実装されています
// See: https://github.com/owner/repo/blob/abc123/docs/api-spec.md#L15-L30
fn validate_request(req: &Request) -> Result<(), Error> {
    // ...
}
```

## 解決する課題

1. **リンクの陳腐化**: 仕様書が更新されると、コード内のパーマネントリンクが古い仕様を指し続ける
2. **仕様変更の検出漏れ**: 仕様が変わったのに実装が追従していないことに気づけない
3. **更新の手間**: リンクを最新化する作業が煩雑
4. **実装との乖離**: 仕様変更に伴い、実装自体の修正が必要になる場合がある

## ユースケース

### 1. CI での仕様乖離チェック

**シナリオ**: チームで開発中、仕様書が更新されたが実装が追従していない状態を検出したい

```yaml
# .github/workflows/docref.yml
name: Check Specification Links
on: [push, pull_request]

jobs:
  check:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # 全履歴を取得（差分比較に必要）
      - name: Install docref
        run: cargo install docref
      - name: Check specification links
        run: docref check --exit-code
```

### 2. 仕様変更時の影響範囲確認

**シナリオ**: 仕様書を変更する前に、どのコードが影響を受けるか把握したい

```bash
# 特定の仕様ファイルを参照しているコードを検索
docref list --filter "docs/api-spec.md"

# 出力例:
# src/api.rs:12      -> docs/api-spec.md#L15-L30
# src/handler.rs:45  -> docs/api-spec.md#L50-L65
# src/client.rs:100  -> docs/api-spec.md#L80-L95
```

### 3. リンクの定期メンテナンス

**シナリオ**: 開発の節目で、古くなったリンクをまとめて最新化したい

```bash
# まず確認
docref check --show-diff

# 問題なければ更新
docref update
```

### 4. 仕様変更に伴う実装修正

**シナリオ**: 仕様が変わったので、AI の支援を受けて実装を修正したい

```bash
# 仕様変更を検出し、実装の修正案を提示
docref sync --interactive
```

## 想定ユーザー

- 仕様書駆動で開発を行うチーム
- コードと仕様の一貫性を重視するプロジェクト
- 外部リポジトリの仕様書を参照して実装するライブラリ開発者

## 将来の拡張案

1. **双方向リンク管理**: 仕様書側にも実装へのリンクを自動生成
2. **VS Code 拡張**: エディタ内でリンクの状態を可視化、ホバーで仕様内容表示
3. **仕様カバレッジ**: どの仕様がコードで参照されているかのレポート
4. **Watch モード**: ファイル変更を監視してリアルタイムチェック
