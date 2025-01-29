# stickee Larastan config

Provides a [Larastan](https://github.com/larastan/larastan) config for stickee projects.

Larastan is a [PHPStan](https://phpstan.org/) wrapper for Laravel.

It is ran by using the `phpstan` command and so will be referred to as PHPStan from now on.

## NOTE

**This version of the config will only work for Larastan v2 which is for Laravel versions 9 and above.**

## Installation

```shell
composer require --dev stickee/larastan-config
cp vendor/stickee/larastan-config/dist/phpstan.dist.neon phpstan.dist.neon
```

You must commit the `phpstan.dist.neon` config file.

## Usage

```shell
vendor/bin/phpstan analyse -c phpstan.dist.neon
```

[You should always analyse the whole project.](https://phpstan.org/blog/why-you-should-always-analyse-whole-project)

### Overrides

You may override any of the settings by editing the `phpstan.dist.neon` file.

The options are available at https://phpstan.org/config-reference.

### Baseline

It would be a pain to add PHPStan to your project and have to fix all the existing errors before you can start using it. 
For this reason you can generate a "baseline" with this command:

```shell
vendor/bin/phpstan analyse -c phpstan.dist.neon --generate-baseline
```

and commit the new `phpstan-baseline.neon` file.

This means PHPStan will ignore any errors in this file so you can use PHPStan to check for errors in any new code you add.

If you get any free time you can refer to this file for code that should be fixed and regenerate the baseline (with the same command) afterwards.

### Pre-Commit Hook

You can use PHPStan in combination with [Husky](https://typicode.github.io/husky/) to run during the pre-commit stage.

#### Installation

- install [Husky](https://typicode.github.io/husky/#/?id=automatic-recommended) into your project
- `cp vendor/stickee/larastan-config/dist/.husky/pre-commit .husky/pre-commit`

## CI

An example GitHub actions workflow is included at `/dist/.github/workflows/phpstan.yaml`.

It will run PHPStan against a PR as a "check" and output any errors it finds against the commit that failed.

The action first checks if any PHP files have been changed and if it needs to run at all. This is because PHPStan must analyse all of the application code at once and therefore takes a bit of time, so it's good to skip it if we can.

The action refers to a CI config at `/dist/.github/workflows/phpstan.ci.neon` (that you can copy into the root of your project) which includes the original config and also ignores unmatched ignored errors to keep the check clean of these errors.

## Problems running PHPStan

The following are some of the easily fixable problems you may run into using PHPStan and Larastan:

### Access to an undefined property

An error such as `Access to an undefined property Illuminate\Database\Eloquent\Model::$subscriber_id` means that PHPStan did not properly understand the class of the variable it read.

If you hover over the variable your editor will probably also not be able to understand what it is.

In this example you must provide an inline type-hint:

```diff

+ /** @var Customer $customer */
  $customer = $request->user();
  $customerService = CustomerService::make($customer->subscriber_id);
```

### Package conflicts

PHPStan expects to be installed into the root of your project instead of a subdirectory like we're doing here.

Whilst this means there _should_ be less of a chance of a package conflict, if one does happen it can be harder to diagnose.

For instance if you get this error:

```
PHP Fatal error:  Declaration of Maatwebsite\Excel\Cache\MemoryCache::get($key, $default = null) must be compatible with Psr\SimpleCache\CacheInterface::get(string $key, mixed $default = null): mixed in /home/paul/projects/asda/vendor/maatwebsite/excel/src/Cache/MemoryCache.php on line 62

   Symfony\Component\ErrorHandler\Error\FatalError

  Declaration of Maatwebsite\Excel\Cache\MemoryCache::get($key, $default = null) must be compatible with Psr\SimpleCache\CacheInterface::get(string $key, mixed $default = null): mixed

  at vendor/maatwebsite/excel/src/Cache/MemoryCache.php:62
     58▕
     59▕     /**
     60▕      * {@inheritdoc}
     61▕      */
  ➜  62▕     public function get($key, $default = null)
     63▕     {
     64▕         if ($this->has($key)) {
     65▕             return $this->cache[$key];
     66▕         }
```

then you can fix it by [forcing `psr/simple-cache` to version 2](https://github.com/SpartnerNL/Laravel-Excel/issues/3758#issuecomment-1276743482).

_However_ you need to do this inside of `composer.json` instead of the composer file in the root of your project:

```json
{
    "require": {
        "psr/simple-cache": "^2.0",
        "stickee/larastan-config": "^1.0"
    }
}
```