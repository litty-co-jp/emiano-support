<img src="https://ps.w.org/emiano-pdf-summarizer/assets/icon-256x256.png?rev=3770790" alt="" width="50">

# EMIANO PDF Summarizer

## API キーを wp-config.php に書く

API キーは、プラグインの設定画面に保存する代わりに、`wp-config.php` に書くこともできます。

```php
define( 'EMIANO_GEMINI_API_KEY', 'your-api-key' );
define( 'EMIANO_OPENAI_API_KEY', 'your-api-key' );
define( 'EMIANO_CLAUDE_API_KEY', 'your-api-key' );
```

使う AI 会社の行だけを書きます。

- `wp-config.php` の値が空でなければ、設定画面の値より優先されます。
- `wp-config.php` に移すと、データベースに保存されていたその会社のキーは自動で消されます。

`wp-config.php` は、公開リポジトリなどにキーごと載せないようご注意ください。


## 有料版（PRO）

PROでは、クラシックエディタへの対応や、追加プロンプトの入力・保存が利用できます。

[料金の確認・購入はこちら →](https://checkout.freemius.com/plugin/41083/plan/71246/)
