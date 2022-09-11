# 自动加载

如果你的项目没有实现自动加载，可以利用composer生成一个，步骤如下。

**1. 更改composer.json**  
在composer.json里autoload.psr-4 加入 `"" : "./"` ，例如
```
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

**2. 生成autoload.php**  
执行 `composer dumpautoload`

**3. 加载autoload.php**  

在项目start.php头部加载`vendor/autoload.php` 例如
```
<?php
require_once __DIR__ . '/vendor/autoload.php';
```

**4. 重启workerman**  
`php start.php restart`
