---
title: Cache Utility
description: Cache Utility Guide for the OriginPHP Framework
extends: _layouts.documentation
section: content
---
# Cache

OriginPHP supports `Apcu`, `Memcached` and `Redis` out of the box, you can also use file caching for
large objects or time consuming generating content.

In your bootstrap file you will the configuration for cache. The caching library can work with multiple configurations and engines at the same time. 

Once the configuration is out the way, using the cache is pretty straightforward.

## Caching

### Write

To add an item to the cache.

```php
Use Origin\Cache\Cache;

$success = Cache::write('key',$value);
```

### Read

To read an item from the cache, if it does not find an item it will return `false`

```php
Use Origin\Cache\Cache;

$value = Cache::read('key');
```

### Exists

To check wether a key exists in the cache

```php
Use Origin\Cache\Cache;

if(Cache::exists('key')){
    $bool = Cache::read('key');
}
```

### Delete

Items are automatically deleted based upon the duration setting in the configuration, however if you want
to delete an item manually then use the delete method.

```php
Use Origin\Cache\Cache;

Cache::delete('key');
```


### Clearing the Cache

```php
Cache::clear();
```

### Enabling and disabling the cache

Sometimes you will need to disable the cache, when you disable we switch the engine to the `NullEngine` and your program can work as normal.

```php
Cache::disable();
Cache::enable();
```

### Working with Multiple Configurations

Whether you are using multiple caching engines, or you multiple configurations for a single cache engine (e.g. short duration and long duration caches), the Cache utility is flexible.

You can get the configured Cache store:

```php
$cache = Cache::store('long-duration');
$value = $cache->read('My.key');
```

Or you can pass an options array telling the Cache object which configuration to use


```php
$value = Cache::read('My.key',[
     'config' => 'long-duration'
     ]);
```

## Engines

In all these examples,we are only configuring the default configuration, you can set different configuration names instead of default. When you use the caching functions the default configuration is used by default unless you say otherwise.

The configuration for Cache can be found in `config/cache.php`.

### File Engine

```php
Cache::config('default', [
     'engine' => 'File',
     'file' => LOGS . '/application.log',
     'duration' => '+60 minutes', // string or number of seconds e.g. 3600,
     'prefix' => 'cache_'
     'serialize' => true // set to false if you going to cache large strings such as output which dont need serialization
     'mode' => 0664
]);
```

### Apcu Engine

```php
Cache::config('default', [
    'engine' => 'Apcu',
    'duration' => '+60 minutes', // string or number of seconds e.g. 3600,
    'prefix' => 'cache_'
]);
```

### Memcached Engine

This is a simple configuration for using Memcached.

```php
Cache::config('default', [
     'engine' => 'Memcached',
     'host' => '127.0.0.1',
     'port' => 11211,
     'duration' => '+60 minutes', // string or number of seconds e.g. 3600,
     'prefix' => 'cache_'
]);
```

If your Memcached server is configured with username and password then

```php
Cache::config('default', [
     'engine' => 'Memcached',
     'host' => '127.0.0.1',
     'port' => 11211,
     'username' => 'james',
     'password' => 'secret',
     'duration' => '+60 minutes', // 3600,
     'prefix' => 'cache_'
]);
```

If you are going to use socket then instead of setting host and port, then set the `path` key with the location
of the socket.

```php
Cache::config('default', [
     'engine' => 'Memcached',
     'path' => '/var/sockets/memcached'
]);
```

You can also make connections persistent by setting the `persistent` key to true, or a string which will be the persistent id.

Memcached supports server pools, if you are going to use them then set an array using the `servers` key instead of host and port. The array should be compatible with [memcached addservers](http://php.net/manual/en/memcached.addservers.php).

### Redis Engine

This is a simple configuration for using Redis.

```php
Cache::config('default', [
     'engine' => 'Redis',
     'host' => '127.0.0.1',
     'port' => 6379,
     'duration' => '+60 minutes', // string or number of seconds e.g. 3600,
     'timeout' => 0,
     'prefix' => 'cache_'
]);
```

If your Redis server is configured with a password then

```php
Cache::config('default', [
     'engine' => 'Redis',
     'host' => '127.0.0.1',
     'port' => 6379,
     'password' => 'secret',
     'duration' => 3600, // duration can also be string e.g. +60 minutes
     'prefix' => 'cache_'
]);
```

If you are going to use socket then instead of setting host and port, then set the `path` key with the location of the socket.

```php
Cache::config('default', [
     'engine' => 'Redis',
     'path' => '/var/sockets/redis',
]);
```

You can also make connections persistent by setting the `persistent` key to true, or a string which will be the persistent id.

```php
Cache::config('default', [
     'engine' => 'Redis',
     'host' => '127.0.0.1',
     'port' => 6379,
     'persistent' => 'my-app',
     'duration' => 3600, // duration can also be string e.g. +60 minutes
     'timeout' => 0,
     'prefix' => 'cache_'
]);
```

### Custom Engine

If you want to work with a different backend, it easy to create your own. When configuring cache, instead of passing the `engine` key, use `className` and include the full namespace. 

```php
Cache::config('default', [
    'className' => 'App\Cache\CustomEngine',
    'duration' => 3600,
    'prefix' => 'cache_'
     ]);
```

```php
namespace App\Cache;
use Origin\Cache\Engine\BaseEngine;
class CustomEngine extends BaseEngine
{

}
```

## Installing Cache Engines

The command line instructions have been tested with Ubuntu 20.4.

### Apcu

Apcu is already installed in the Docker container, however if you need to install this manually.

```bash
$ sudo apt-get update
$ sudo apt-get install php-apcu
$ sudo echo 'apc.enable_cli=1' >>  /etc/php/7.4/cli/php.ini
```

### Memcached

See [Dockerized Development Environment](/docs/development/dockerized-development-environment), on how to setup `memcached` in your Docker container.

**To install Memcached on a Ubuntu/Debain based server**

```bash
$ sudo apt-get update
$ sudo apt-get install memcached
$ sudo apt-get install php-memcached
```

### Redis

See [Dockerized Development Environment](/docs/development/dockerized-development-environment), on how to use `Redis` in your Docker container.

**To install Redis on a Ubuntu/Debain based server**

```bash
$ pecl install redis
$ sudo echo 'extension=redis.so' >> /etc/php/7.4/apache2/php.ini
$ sudo echo 'extension=redis.so' >> /etc/php/7.4/cli/php.ini
```