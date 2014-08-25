Composer
===========

Installation
---------------

- Get Composer

```
% curl -sS https://getcomposer.org/installer | php

#!/usr/bin/env php
All settings correct for using Composer
Downloading...

Composer successfully installed to: /Users/wnoguchi/Documents/tmp/phalcon/composer.phar
Use it: php composer.phar
```

- Install dependencies

```
php composer.phar install
```

- Using composer autoloader

```php
require 'vendor/autoload.php';
```

References
------------

1. [Composerを使ってみた - 戦場のプログラマー](http://blog.pg1x.com/entry/2013/12/07/201728)
1. [Getting Started](https://getcomposer.org/doc/00-intro.md)
1. [Basic Usage](https://getcomposer.org/doc/01-basic-usage.md)
