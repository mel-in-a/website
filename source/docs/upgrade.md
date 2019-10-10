---
title: Upgrade Guide
description: Upgrade Guide for 2.0
extends: _layouts.documentation
section: content
---
# Upgrade Guide

## Welcome

Version 2 removes deprecated features and provides a more organized folder structure and makes it easier to work with given the number of folders that are now used. 

Other changes such as strict, return types, dropping public properties and other changes to take advantage of modern PHP features and best practices. 

`Model` and `Controller` callbacks have been redesigned to be registered and work with `Concerns` in a powerful way.

I have been working full time on the framework to get this where it is now, changes going forward from here should be slow, with a focus on improving code base, developing and testing with future PHP versions, bug and security fixes.

I will be happy to help you with upgrading if you contact me within the following weeks, just send me an email to `js@originphp.com`.

## Not Upgrading

Firstly, if you don't want to upgrade to the new version then you will need to update your `composer.json` file, to ensure that when you run `composer update` it does not download version 2.x, the `composer.json` file from most versions of the framework will just update to the latest version automatically.

> Very early versions of the framework might trigger errors when upgrading to later versions

```json
  "require": {
    "php": "^7.2.0",
    "originphp/framework": "^1.30"
  },
  "require-dev": {
    "originphp/debug-plugin": "1.3.3",
    "phpunit/phpunit": "^7.5"
  }
```

## Composer.json

The composer.json file has been changed, this now respects versions.

```json
{
  "name": "originphp/app",
  "description": "OriginPHP Application",
  "homepage": "https://www.originphp.com",
  "type": "project",
  "license": "MIT",
  "require": {
    "php": "^7.2.0",
    "originphp/framework": "^2.0"
  },
  "require-dev": {
    "originphp/debug-plugin": "^1.4",
    "phpunit/phpunit": "^8.0"
  },
  "minimum-stability": "dev",
  "prefer-stable": true,
  "config": {
    "preferred-install": "dist",
    "optimize-autoloader": true,
    "sort-packages": true
  }
}
```

## Front Controller

The front controller `public/index.php` changed to easily allow for changes to be carried out in the framework, if needed. See [Github](https://github.com/originphp/app/blob/master/public/index.php).

## Bootstrap

The loading of configuration files has been moved from the framework bootstrap to the application bootstrap `config/bootstrap.php`.

For example

```php
include 'application.php';
include 'log.php';
include 'cache.php';
include 'database.php';
include 'storage.php';
include 'email.php';
include 'queue.php';
```

## Parent Class Changes

Parent classes such as `AppController` have been renamed to use the Application prefix, e.g. `ApplicationController`.

- AppController
- AppModel
- AppService
- AppMailer
- AppJob
- AppHelper

## New Classes

### Console/Application.php
Console now introduces its own Application.php, this will allow for adding shared features and configuration to console commands in the future.

Here is the contents of `app/Console/Application.php`

```php
namespace App\Console;
use Origin\Console\BaseApplication;
class Application extends BaseApplication
{
}
```

### Exception/ApplicationException

A new folder called exception has been added, and the class `ApplicationException` has been added.

Here is the contents of `app/Exception/ApplicationException.php`

```php
namespace App\Exception;
use Origin\Exception\Exception;
class ApplicationException extends Exception
{
}
```

## Folder Structure Changes

HTTP and Console have been separated, as a result views for Http are in the HTTP folder and email templates are stored in the Mailer folder.

```
-- app
    |-- Console
    |   |-- Command
    |-- Http
    |   |-- Controller
    |   |-- Middleware
    |   |-- View
    |-- Job
    |-- Mailer
    |   |-- Template
    |   |-- Layout
    |-- Model
```

Moving the files to new folder structure will require references to these to be changed as below:

- `App\Command` change to `App\Console\Command`
- `App\Controller` change to `App\Http\View`
- `App\View` change to `App\Http\View`
- `App\Middleware` change to `App\Http\Middleware`

## Mailer Templates

Templates for Mailers are now stored in a different location and use a different filename structure

`/app/Mailer/Template/welcome_email.html.ctp`
`/app/Mailer/Template/welcome_email.text.ctp`

Layouts for mailers location have also changed

`app/Mailer/Layout/mailer.ctp`.

Create the default layout `app/Mailer/Layout/default.ctp`

```php
<!DOCTYPE html>
<html>
  <head>
    <meta content='text/html; charset=UTF-8' http-equiv='Content-Type' />
  </head>
  <body>
     <?= $this->content() ?>
  </body>
</html>
```

## Class Location Changes

As a result of folder structure changes framework classes have been moved following the same structure

Framework Locations

- `Origin\Command\Command` changed to `Origin\Console\Command\Command`
- `Origin\Controller\Controller` changed to `Origin\Http\Controller\Controller`
- `Origin\Controller\Component\Component` changed to `Origin\Http\Component\Component`
- `Origin\View\View` changed to  `Origin\Http\View\View`
- `Origin\Http\Middleware` changed to `Origin\Http\Middleware\Middleware`


All HTTP exceptions for example:

- `Origin\Exception\NotFoundException` changed to `Origin\Exception\NotFoundException`

These include BadRequest,Forbidden,HttpException,InternalError,MethodNotAllowed, ServiceUnavailable, NotImplemented.


## Controller Callbacks

`beforeFilter` and `afterFilter` have been renamed to `startup` and `shutdown` respectively, these do not need to be registered. 

You can register additional callbacks which are called between these, using the following methods

```php
$this->beforeAction('checkRequest');
$this->afterAction('cleanResponse');
$this->beforeRedirect('logSomething');
$this->beforeRender('cacheView');
```

If you have used `beforeRedirect` or `beforeRender` rename these, and then register as above.

## Connection

`Model::$datasource` has been renamed to `Model::$connection`, if you have used this feature then you will need
change this.

Console commands with the `datasource` option has been renamed to `connection` or short option `c`, if you are using these in scripts or cron jobs then you will need to adjust.

## Model Callbacks

Model callbacks have been changed completely, the return types and and the arguments are both the entity and an `ArrayObject` with the option. With the exception, `beforeFind` and `afterfind`. The `afterfind` first argument is always a `Origin\Model\Collection`, even if you are doing find first.

Callbacks are now registered but use the same name for registration so any callbacks that have used you will need to rename the existing callback, by adding a callbacks suffix, then just defining it. For example:

```php
public function initialize(array $config)
{
  $this->beforeFind('beforeFindCallback');
}

public function beforeFindCallback(ArrayObject $options) : bool
{
    return true;
}
```

See [Callbacks guide](https://www.originphp.com/docs/model/callbacks/) for more information.

## Configure Class

The `Configure` class has been renamed to `Config`, depending upon the version you used to create your project you may have already been using this. You should your check your `config/application.php` file.

## Return Types

`Initialize` and `execute` methods have a return type of void, except in Service Objects where the execute method should return nothing or a `Result` object.

In your PHPUnit tests `startup` and `shutdown` also have a return type of void.

## Public Properties

All public properties have been changed to protected, including `Mailer`,`Job`, `Schema` which previously used public properties for configuration.

## Middleware

Previously Middleware used `startup` and `shutdown` as aliases for `invoke` and `process`, this has been changed, and startup and shutdown now callbacks inline with rest of framework. Rename your Middleware `startup` to `invoke` and `shutdown` to `process`.

## Behaviors

### Framework References

In your models any references to old framework behaviors need to be adjusted to their equivalent as a `Concern`. 

```php
namespace App\Model;

use Origin\Model\Model;
use Origin\Model\Concern\Delocalizable;
use Origin\Model\Concern\Timestampable;

class ApplicationModel extends Model
{
    use Timestampable,Delocalizable;
    public function initialize(array $config): void
    {
    }
}
```

### Custom Behaviors

If you have created any custom Behaviors, these will be need to converted to `Concerns`, the code will be almost identical, except you dont need to reference `$this->model` since a `Concern` is a trait.


## Removed Features

All previous deprecated features have been removed, the following functions have also been removed

- Behaviors
- Text::random
- uuid
- left
- right
- contains
- replace

## Other Changes

- Security::uid returns base 62 string with a length of 15. e.g `64cjBxfz2JPhyCQ`

## PHPUnit

The framework has been updated to work with PHPUnit 8.3+. 

## Migations

The [migration schema](https://github.com/originphp/app/blob/master/database/migrations.php) has changed, version is now a `bigint` field. You will need to reload the schema into your database.

## Cookies

When writing cookies, setting the expiration is done via the options array, now the third argument, in any class that allows you to write cookies.

```php
$this->response->cookie('key','value',['expires'=>'+ 10 days']);
$this->Cookie->write('key','value',['expires'=>'+ 10 days']);
```

## Initialization Trait

If you have used the Initialization trait, then you will need to change the initialization method name, which now must include the prefix initialize

```php
trait  Foo
{
  protected function initializeFoo()
  {

  }
}
```

## Note

Log out of your existing application and delete any cookies to prevent possible errors.