# EMIANO Analytics Ranking

## 接続の設定

GA4 にはサービスアカウントの鍵（JSON）で接続します。設定画面は「設定」→「Analytics Ranking」にあります（ほかの EMIANO プラグインも有効なときは「EMIANO」→「Analytics Ranking」）。

1. Google Cloud でプロジェクトを作り、Google Analytics Data API と Google Analytics Admin API を有効にします。
2. サービスアカウントを作り、JSON 形式の鍵を作成します。
3. GA4 の「プロパティのアクセス管理」で、サービスアカウントのメールアドレスを「閲覧者」として追加します。
4. 設定画面に、GA4 の「プロパティの詳細」に出ているプロパティ ID と、鍵の JSON を入力して保存します。
5. 「Test connection」を押して、接続できるか確かめます。接続テストは保存済みの設定を使うので、先に保存してください。

## ランキングを表示する

**Analytics Ranking** ブロック、またはショートコードで表示します。

### ショートコード

```text
[emiano_ranking period="1w" post_type="post"]
```

| 属性 | 値 | 省略したとき |
|---|---|---|
| `period` | 設定画面で選んだ期間：`12h`、`1d`、`3d`、`1w`、`1m`、`3m`、`6m` | 選んだ期間のうち最も短いもの |
| `post_type` | 公開の投稿タイプを1つ | `post` |
| `category` | カテゴリのスラッグを1つ。子カテゴリの記事も含みます | すべてのカテゴリ |

カテゴリ別のランキングは、全体のランキングの中からそのカテゴリの記事を選んで並べます。記事の少ないカテゴリでは、10件に届かないことがあります。

設定画面で選んでいない期間を指定すると、データがないときと同じ表示になります。

テーマのテンプレートに書くときは、次のようにします。

```php
<?php echo do_shortcode( '[emiano_ranking period="1w"]' ); ?>
```

### ブロック

ブロックの設定パネルで、期間、投稿タイプ、カテゴリを選びます。

### 共通の設定

サムネイル、順位、投稿日、最終更新日時、タイトル、データがないときの文言は、ブロックやショートコードごとではなく、設定画面の「Display」でサイト全体に設定します。

### テーマやプラグインから使う

ランキングの記事 ID を取得する関数と、表示を変えるフィルターがあります。

```php
// 記事 ID の配列（順位の順、最大10件）。引数はショートコードと同じです。
$ids = emiano_ar_get_ranking( array( 'period' => '1w', 'category' => 'news' ) );
```

| フィルター | 引数 | 用途 |
|---|---|---|
| `emiano_ar_ranking_post_ids` | `$ids`（候補の記事 ID、順位の順）、`$args` | 記事の除外・並び替え。投稿タイプとカテゴリで絞る前に適用されます |
| `emiano_ar_ranking_html` | `$html`、`$post_ids`（表示した記事 ID）、`$args` | ショートコードとブロックの HTML を変更。データがないときの文言には適用されません |

```php
// 例：ID 123 の記事をランキングから除く
add_filter( 'emiano_ar_ranking_post_ids', function ( $ids ) {
	return array_diff( $ids, array( 123 ) );
} );
```

## ランキングが更新されないとき

ランキングは、WordPress の予約実行（WP-Cron）で GA4 から定期的に取得します。表示のたびには取得しません。

### 取得の状況を確かめる

設定画面の「Data」タブにある取得状況の表を見ます。

- 「Result」が失敗で「Error」に内容が出ているときは、接続の設定を見直してください（「[接続の設定](#接続の設定)」）。
- 「Next retrieval」の日時を過ぎても取得されないときは、WP-Cron が動いていない可能性があります。「ツール」→「サイトヘルス」に、予定されたイベントの遅れや、ループバックリクエストの失敗が出ていないか確かめてください。

### WP-Cron が動かないときの対処

WP-Cron はサイトへのアクセスをきっかけに動きます。アクセスの少ないサイトでは、取得が遅れることがあります。

次のような場合は、WP-Cron が動かないことがあります。

- `wp-config.php` で `DISABLE_WP_CRON` を設定している
- ベーシック認証やセキュリティの設定で、サイトが自分自身へ送る通信（ループバックリクエスト）が止められている

対処は、次のどちらかです。どちらも WordPress の標準の方法で、このプラグイン専用の設定ではありません。

- **サーバーの cron から `wp-cron.php` を呼ぶ**：レンタルサーバーの cron 設定で、定期的に `https://（サイトの URL）/wp-cron.php` を呼び出します。設定の方法はサーバーによって違います。
- **`ALTERNATE_WP_CRON` を使う**：`wp-config.php` に `define( 'ALTERNATE_WP_CRON', true );` を追加します。ループバックリクエストが使えないサイト向けの方法です。

## Simple GA Ranking／Simple GA4 Ranking からの乗り換え

### 接続設定

接続設定は引き継げません。Simple GA4 Ranking は OAuth（クライアント ID・クライアントシークレット）で接続しますが、このプラグインはサービスアカウントの鍵で接続します。

- GA4 のプロパティ ID は同じものを使えます。
- OAuth のクライアントは使いません。サービスアカウントを新しく作り、GA4 に閲覧者として追加します。手順は「[接続の設定](#接続の設定)」を見てください。

### ショートコードの書き換え

`[sga_ranking]` を `[emiano_ranking]` に書き換えます。

| Simple GA (4) Ranking | このプラグイン |
|---|---|
| `[sga_ranking]` | `[emiano_ranking]` |
| 設定画面の「何日前までのランキングを表示するか」 | 設定画面で期間を選び、`period` 属性で指定（例：7日 → `1w`、30日 → `1m`） |
| 設定画面の「表示数」 | 10件で固定 |
| `post_type`（カンマ区切りで複数） | `post_type`（1つだけ。固定ページは指定できません） |
| `exclude_post_type` | なし |
| `category__in` | `category`（カテゴリのスラッグを1つ） |
| `post_tag__in`、`category__not_in` など、そのほかの分類での絞り込み | なし |
| `sga_ranking_ids()`、`sga_ranking_get_date()` | `emiano_ar_get_ranking()`（記事 ID だけを返します。PV は返しません） |
| `sga_ranking_exclude_post` | `emiano_ar_ranking_post_ids` で除外 |
| `sga_ranking_shortcode` | `emiano_ar_ranking_html` |
| ウィジェット「Simple GA 4 Ranking」 | **Analytics Ranking** ブロック |

出力される HTML とクラス名（`sga-ranking` → `emiano-ar-ranking`）が違うため、テーマ側で見た目を調整している場合は CSS の書き換えが必要です。
