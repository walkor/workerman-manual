＃ 自動ロード

自動ロードが実装されていない場合、composerを使用して生成することができます。手順は次のとおりです。

**1. composer.jsonの変更**  
composer.jsonにautoload.psr-4に```"" : "./"```を追加します。例えば
```json
{
    "require": {
        "workerman/workerman" : "^v4.0.0"
    },
    "autoload": {
        "psr-4": {
            "" : "./"
        }
    }
}
```

**2. autoload.phpの生成**  
`composer dumpautoload`を実行します

**3. autoload.phpのロード**  
プロジェクトのstart.phpの先頭で`vendor/autoload.php`をロードします。例えば
```php
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. workermanの再起動**  
`php start.php restart`
