# Advanced PHP Heroku Build Pack

### What makes it unique?

* Supports PHP 5.3, 5.4 and 5.5
* Uses the memory of the dyno more efficiently by going with NGINX and PHP-FPM instead of Apache/mod_php
* Supports Composer out of the box
* No writing NGINX configuration files with a simple configuration driven by your `composer.json`.
* Zero-Configuration Opencart deployment.
* Dynamic installing of [supported extensions](support/ext) listed as `ext-` requirments in `composer.json`.

### How to use it

Use the `--buildpack` parameter when creating a new app:

    heroku create --buildpack https://github.com/mithereal/heroku-buildpack-opencart myapp

Or set the `BUILDPACK_URL` config var on an existing app:

    heroku config:set BUILDPACK_URL=https://github.com/mithereal/heroku-buildpack-opencart

* * *

If you want to be on the bleeding edge and use pre-release features, then use
`git://github.com/mithereal/heroku-buildpack-opencart#development` as buildpack
url.

### Stack

* NGINX 1.5 or 1.6
* PHP 5.3, 5.4 and 5.5, with [ZendOpcache][] and [APCu][] ([Info](https://sample-php-base.appsdeck.eu))
* PHP-FPM

[ZendOpcache]: http://pecl.php.net/package/ZendOpcache
[APCu]: http://pecl.php.net/package/apcu
[Available PHP Versions]: http://opencart-heroku-buildpack-php.s3.amazonaws.com/manifest.php
[Available NGINX Versions]: http://opencart-heroku-buildpack-php.s3.amazonaws.com/manifest.nginx

### Detection

This buildpack detects apps when the app has a `composer.lock` in the
app's root.

When `system\startup.php` is detected in the app's root, every ".php" file is served with PHP,
and the document root is set to the app root.

When a `composer.lock` is detected, then the buildpack does `composer
install --no-dev`.

### Environment

This buildpack sets environment variables during compile and runtime:

* `HEROKU_BUILD_TIME`: Time when the slug was compiled. Format is `%Y%m%d%H%M%S`, e.g. `20131103111548`

This buildpack also detects when the app has a node `package.json` in the
app's root. And will install node dependencies like less for example.

### Extensions

When the buildpack encounters `ext-` requirements in your `composer.json`, it will look
up the extension name in the [supported extensions](support/ext) and install them.

The version constraint is ignored currently.

For example, to install the Sundown extension:

```
{
    "require": {
        "ext-sundown": "*"
    }
}
```

Note that the extension requirements defined by dependencies are not taken into account there.
It must be required by the project itself.

### Logging

This buildpack defines default log files by framework.
It also defines log files nginx and php.

### Configuration

Configuration is done via a file named `composer.json` in the app's
root.

A simple configuration could look like this:

    {
        "require": {
            "php": ">=5.4.0",
            "silex/silex": "~1.0@dev"
        },
        "extra": {
            "heroku": {
                "document-root": "web",
                "index-document": "index.php"
            }
        }
    }

This configures an app with the document root set to the project's `web`
directory, and sets that all requests should go through `web/index.php`
which contains the application's front controller.

### Configuration Directives

This buildpack supports configuration through directives placed in the `heroku`
key in the `extra` object.


#### document-root

Document root relative to the app root. Defaults to the app root.

    "document-root": "web"

#### index-document

_Default: "index.php"_

Index Document relative to the document root.

    "index-document": "app.php"

#### engines

Set PHP and NGINX versions.

To launch the app with PHP 5.3.23 and NGINX 1.3.14:

    "engines": {
        "php": "5.3.23",
        "nginx": "1.3.14"
    }

Set the version to "default" to use the current default version. The current
default versions are NGINX `1.4.4` and PHP `5.5.10`.

The version identifiers can also include wildcards, e.g. `5.4.*`. At the
time of writing, PHP `5.4.26` would be used in this case. This also
works for NGINX.

When a file named `.php-version` exists in the project root, then the
PHP version is read from this file instead.

See also:

* [Available NGINX Versions][]
* [Available PHP Versions][]

#### php-config

_Default: []_

Add directives to the `php.ini`.

    "php-config": [
        "display_errors=off",
        "short_open_tag=on"
    ]

#### php-includes

_Default: []_

Include additional .ini files that should be parsed after the default php.ini. File paths
are treated relative to the app root.

Example:

    "php-includes": ["etc/php.ini"]

#### nginx-includes

_Default: []_

Include additional config files into the NGINX configuration. Config
files are included into the `server` scope and are loaded after the
framework provided config. File paths are treated relative to the app
root.

Example:

    "nginx-includes": ["etc/nginx.conf"]

#### compile

_Default: []_

Run console commands on slug compilation.

    "compile": [
        "php app/console assetic:dump --env=prod --no-debug"
    ]

_Note: pecl is not runnable this way._

#### newrelic

_Default: false_

Enable instrumentation support via [New Relic](http://newrelic.com).
It's recommended to add the New Relic addon to your Heroku app, but you
can also set your license key manually by setting the `NEW_RELIC_LICENSE_KEY` config var via `heroku config:set`.

    "newrelic": true

#### log-files

_Default: []_

The buildpack defines default log files by framework and some log files for php-fpm and nginx.
Any file put in `log-files` will be be appended to the list.
A tail on each unique log file will be run at application startup

    "log-files": [
        "app/logs/rabbit-mq.log",
        "vendor/nginx/stuff.log"
    ],



## Contributing

Please see the [CONTRIBUTING](/CONTRIBUTING.md) file for all the
details.
