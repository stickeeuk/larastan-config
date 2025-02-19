# stickee Larastan config

Provides a [Larastan](https://github.com/larastan/larastan) config for stickee projects.

Larastan is a [PHPStan](https://phpstan.org/) wrapper for Laravel.

It is ran by using the `phpstan` command and so will be referred to as PHPStan from now on.

## Installation

```shell
composer require --dev stickee/larastan-config
cp vendor/stickee/larastan-config/dist/phpstan.dist.neon phpstan.dist.neon
```

You must commit the `phpstan.dist.neon` config file.

## Setup

See [the upgrade guide on PHPStan](https://github.com/larastan/larastan/blob/3.x/UPGRADE.md#correct-return-types-for-model-relation-methods) for a Rector rule that should be run before using this config.

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

## CI

An example GitHub actions workflow is included at `/dist/.github/workflows/phpstan.yaml`.

It will run PHPStan against a PR as a "check" and output any errors it finds against the commit that failed.

The action first checks if any PHP files have been changed and if it needs to run at all. This is because PHPStan must analyse all of the application code at once and therefore takes a bit of time, so it's good to skip it if we can.

The action refers to a CI config at `/dist/.github/workflows/phpstan.ci.neon` (that you can copy into the root of your project) which includes the original config and also ignores unmatched ignored errors to keep the check clean of these errors.

## Problems running PHPStan

The following are some of the easily fixable problems you may run into using PHPStan and Larastan:

### Access to an undefined property

There is [a guide on the PHPStan blog](https://phpstan.org/blog/solving-phpstan-access-to-undefined-property) that contains suggestions for fixing this.
