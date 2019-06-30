---
title: Logging
description: Logging Guide for the OriginPHP Framework
extends: _layouts.documentation
section: content
---
# Logging

> This is for the old logger which is being deprecated. See [Log](/docs/log.md)

OriginPHP uses a minimialistic file logger based upon the PSR 3 standard.

```php
use Origin\Core\Logger;
$logger = new Logger('MathLib');
$logger->error('Something has gone wrong.');
```
This will produce something like this in `logs/application.log`.

```
[2019-03-10 13:37:49] MathLib ERROR: something has gone wrong.
```

You can also use placeholders in the message.

```php 
$logger->info('Email sent to {email}',['email'=>'donny@example.com']);
```

You can call the following logging methods on the Logger object

- *debug*
- *info*
- *notice*
- *warning*
- *error*
- *critical*
- *alert*
- *emergency*

Sometimes you might want to log to a separate file simply call the `filename` method.

```php 
$logger = new Logger('WeatherController')
$logger->filename('/var/www/logs/other.log');
$logger->warning('Humidity not set');
```