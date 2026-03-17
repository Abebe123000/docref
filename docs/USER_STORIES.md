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

### 1. リンクの陳腐化

仕様書が更新されると、コード内のパーマネントリンクが古い仕様を指し続ける

**現状**: リンクが古くなっていることに気づかず放置される。気づいた開発者が手動でコードを grep して該当箇所を探し、仕様書の git log を見て最新のコミットハッシュに書き換えている。

### 2. 仕様変更への追従漏れ

仕様が変わったのに実装が追従していないことに気づけず、修正も手動で行う必要がある

**現状**: 仕様書の変更は PR やチャットで共有されるが、どのコードへの影響があるかは各自の記憶や属人的な知識に依存する。仕様変更を見落とした実装者がバグを混入させ、レビューやテストで初めて発覚する。気づいた場合も、変更内容の把握・影響範囲の調査・実装修正のすべてを開発者が頭の中で組み立てる必要があり、見落としが起きやすい。

### 3. 更新の手間

リンクを最新化する作業が煩雑

**現状**: 仕様書の特定ファイルを参照しているコードを洗い出すために `grep` を手で実行し、ヒットした行を一つずつ開いてハッシュを書き換えていく。参照箇所が多いと数十ファイルにわたる単純作業になる。


## ユースケース

### 1. リンクの陳腐化への対処

**解決策**: `docref check` で現在のリンクが指す内容と最新の仕様との差分を検出し、`docref update` で一括更新する

```bash
# 古くなったリンクを確認（差分表示）
docref check --show-diff

# src/api.rs:12 のリンクが指す仕様と最新の差分:
# - リクエストのタイムアウトは 30 秒以内とする
# + リクエストのタイムアウトは 10 秒以内とする

# リンクを最新コミットに更新
docref update
```

### 2. 仕様変更への追従漏れへの対処

**解決策**: CI に組み込んで仕様変更を自動検出し、`docref sync` で影響箇所の修正を提案する

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
        # 仕様が変更されていれば非ゼロ終了 → CI が失敗
```

```bash
# 仕様変更を検出し、実装の修正案を提示
docref sync --interactive

# 出力例:
# [src/api.rs:12] 仕様が変更されました
# 変更内容: タイムアウトが 30 秒 → 10 秒
# 修正提案: TIMEOUT_SECS の値を 30 から 10 に変更する
# 適用しますか？ [y/N]
```

### 3. 更新の手間への対処

**解決策**: `docref update` でリポジトリ内のリンクをまとめて自動更新する

```bash
# 影響を受けるリンクの一覧を確認
docref list --filter "docs/api-spec.md"

# 出力例:
# src/api.rs:12      -> docs/api-spec.md#L15-L30 [outdated]
# src/handler.rs:45  -> docs/api-spec.md#L50-L65 [outdated]
# src/client.rs:100  -> docs/api-spec.md#L80-L95 [up-to-date]

# 古いリンクをすべて一括更新
docref update --filter "docs/api-spec.md"
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
