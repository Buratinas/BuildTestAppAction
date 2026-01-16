# Sylius Build Test Application GitHub Action

The goal of this action is to reduce the repetitive part of our every Sylius GitHub Workflow file.

## Usage

Below you can find an example of a workflow file that uses this action. Keep in mind `actions/checkout@v4`
action is required and must be run **before** `SyliusLabs/BuildTestAppAction` action.

```yaml
name: Example Workflow

on:
    push: ~

jobs:
    tests:
        runs-on: ubuntu-latest

        name: "Sylius ${{ matrix.sylius }}, PHP ${{ matrix.php }}, Symfony ${{ matrix.symfony }}, ${{ matrix.database }}"

        env:
            APP_ENV: test_cached

        strategy:
            fail-fast: false
            matrix:
                php: ["8.3"]
                symfony: ["^6.4", "^7.4"]
                sylius: ["~2.1.0", "~2.2.0"]
                database: ["mysql:8.4"]

                include:
                    - php: "8.3"
                      symfony: "^7.4"
                      sylius: "~2.2.0"
                      database: "postgres:16"

        steps:
            -   uses: actions/checkout@v4

            -   name: Build application
                uses: SyliusLabs/BuildTestAppAction@v4
                with:
                    e2e: "yes"
                    database: ${{ matrix.database }}
                    php_version: ${{ matrix.php }}
                    symfony_version: ${{ matrix.symfony }}
                    sylius_version: ${{ matrix.sylius }}

            -   name: Run Behat
                run: vendor/bin/behat
```

The action automatically sets `DATABASE_URL` environment variable with unified credentials (`sylius:sylius@sylius`).
No need to define it in your workflow.

## Inputs

| Name                         | Description                                                                            | Default                                                 | Required |
|------------------------------|----------------------------------------------------------------------------------------|---------------------------------------------------------|----------|
| `build_type`                 | Type of the build. Possible values: `sylius`, `app`, `plugin`.                         | `app`                                                   | No       |
| `application_root_dir`       | Root directory of the application (useful in monorepos).                               | Resolved automatically based on `build_type`.           | No       |
| `application_test_dir`       | Directory of the test application (used to run console commands, fixtures, etc.).      | Resolved automatically based on `build_type`.           | No       |
| `console_path`               | Relative path to Symfony's `console` binary (overrides auto-detection)                 | `vendor/bin/console` or `bin/console` (auto)            | No       |
| `cache_key`                  | A cache key to be used (used with `actions/cache`).                                    | *(empty)*                                               | No       |
| `cache_restore_key`          | A cache restore key to be used.                                                        | *(empty)*                                               | No       |
| `e2e`                        | Whether to prepare the test application for end-to-end tests.                          | `no`                                                    | No       |
| `e2e_js`                     | Whether to prepare the test application for JS-based end-to-end tests.                 | `no`                                                    | No       |
| `database`                   | Database engine and version (e.g., `mysql:8.4`, `postgres:16`, `mariadb:10.11`).       | `mysql:8.4`                                             | No       |
| `php_version`                | PHP version to be used.                                                                | *(none)*                                                | Yes      |
| `php_extensions`             | List of PHP extensions to install.                                                     | `intl, gd, opcache, mysql, pdo_mysql, pgsql, pdo_pgsql` | No       |
| `php_tools`                  | PHP tools to install (e.g. `symfony`, `composer`).                                     | `symfony`                                               | No       |
| `php_coverage`               | Coverage tool to use (e.g. `xdebug`, `pcov`, `none`).                                  | `none`                                                  | No       |
| `symfony_version`            | Symfony version to be used.                                                            | *(none)*                                                | Yes      |
| `sylius_version`             | Sylius version to be used.                                                             | *(none)* – no restriction applied                       | No       |
| `sylius_integration`         | Internal usage: run Sylius with a specific integration.                                | *(empty)*                                               | No       |
| `node_version`               | Node.js version to install (used when `e2e_js: yes`).                                  | `22.x`                                                  | No       |
| `chrome_version`             | Google Chrome version to install (for JS e2e tests).                                   | `stable`                                                | No       |
| `opcache_enable`             | Control OPcache (`yes`=enable with test-optimized settings, `no`=disable).             | *(empty)* – use system defaults                         | No       |
| `composer_minimum_stability` | Set minimum-stability in composer.json (e.g., `stable`, `RC`, `beta`, `alpha`, `dev`). | *(empty)* – no restriction applied                      | No       |
| `composer_prefer_stable`     | Set prefer-stable in composer.json (`yes` or `no`).                                    | *(empty)* – no restriction applied                      | No       |
