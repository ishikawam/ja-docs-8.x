# アップグレードガイド

- [Upgrading To 8.0 From 7.x](#upgrade-8.0)

<a name="high-impact-changes"></a>
## 重要度の高い変更

<div class="content-list" markdown="1">
- [Model Factories](#model-factories)
- [Queue `retryAfter` Method](#queue-retry-after-method)
- [Queue `timeoutAt` Property](#queue-timeout-at-property)
- [Pagination Defaults](#pagination-defaults)
</div>

<a name="medium-impact-changes"></a>
## 重要度が中程度の変更

<div class="content-list" markdown="1">
- [PHP 7.3.0 Required](#php-7.3.0-required)
- [Failed Jobs Table Batch Support](#failed-jobs-table-batch-support)
- [Maintenance Mode Updates](#maintenance-mode-updates)
- [The `assertExactJson` Method](#assert-exact-json-method)
</div>

<a name="upgrade-8.0"></a>
## Upgrading To 8.0 From 7.x

#### アップグレード見積もり時間：１５分

> {note} 私達は、互換性を失う可能性がある変更を全部ドキュメントにしようとしています。しかし、変更点のいくつかは、フレームワークの明確ではない部分で行われているため、一部の変更が実際にアプリケーションに影響を与えてしまう可能性があります。

<a name="php-7.3.0-required"></a>
### PHP 7.3.0 Required

**影響の可能性： 中程度**

The new minimum PHP version is now 7.3.0.

<a name="updating-dependencies"></a>
### 依存パッケージのアップデート

`composer.json`ファイル中に指定されている以下のパッケージ依存を更新してください。

<div class="content-list" markdown="1">
- `laravel/framework` to `^8.0`
- `nunomaduro/collision` to `^5.0`
- `guzzlehttp/guzzle` to `^7.0.1`
- `facade/ignition` to `^2.3.6`
</div>

The following first-party packages have new major releases to support Laravel 8. If applicable, you should read their individual upgrade guides before upgrading:

<div class="content-list" markdown="1">
- [Horizon v5.0](https://github.com/laravel/horizon/blob/master/UPGRADE.md)
- [Passport v10.0](https://github.com/laravel/passport/blob/master/UPGRADE.md)
- [Socialite v5.0](https://github.com/laravel/socialite/blob/master/UPGRADE.md)
- [Telescope v4.0](https://github.com/laravel/telescope/releases)
</div>

Finally, examine any other third-party packages consumed by your application and verify you are using the proper version for Laravel 8 support.

### Collections

#### The `isset` Method

**影響の可能性： 低い**

To be consistent with typical PHP behavior, the `offsetExists` method of `Illuminate\Support\Collection` has been updated to use `isset` instead of `array_key_exists`. This may present a change in behavior when dealing with collection items that have a value of `null`:

    $collection = collect([null]);

    // Laravel 7.x - true
    isset($collection[0]);

    // Laravel 8.x - false
    isset($collection[0]);

### Eloquent

<a name="model-factories"></a>
#### Model Factories

**影響の可能性： 高い**

Laravel's [model factories](/docs/{{version}}/database-testing#creating-factories) feature has been totally rewritten to support classes and is not compatible with Laravel 7.x style factories. However, to ease the upgrade process, a new `laravel/legacy-factories` package has been created to continue using your existing factories with Laravel 8.x. You may install this package via Composer:

    composer require laravel/legacy-factories

#### The `Castable` Interface

**影響の可能性： 低い**

The `castUsing` method of the `Castable` interface has been updated to accept an array of arguments. If you are implementing this interface you should update your implementation accordingly:

    public static function castUsing(array $arguments);

#### Increment / Decrement Events

**影響の可能性： 低い**

Proper "update" and "save" related model events will now be dispatched when executing the `increment` or `decrement` methods on Eloquent model instances.

### Events

#### The `Dispatcher` Contract

**影響の可能性： 低い**

The `listen` method of the `Illuminate\Contracts\Events\Dispatcher` contract has been updated to make the `$listener` property optional. This change was made to support automatic detection of handled event types via reflection. If you are manually implementing this interface, you should update your implementation accordingly:

    public function listen($events, $listener = null);

### Framework

<a name="maintenance-mode-updates"></a>
#### Maintenance Mode Updates

**影響の可能性： 状況による**

The [maintenance mode](/docs/{{version}}/configuration#maintenance-mode) feature of Laravel has been improved in Laravel 8.x. Pre-rendering the maintenance mode template is now supported and eliminates the chances of end users encountering errors during maintenance mode. However, to support this, the following lines must be added to your `public/index.php` file. These lines should be placed directly under the existing `LARAVEL_START` constant definition:

    define('LARAVEL_START', microtime(true));

    if (file_exists(__DIR__.'/../storage/framework/maintenance.php')) {
        require __DIR__.'/../storage/framework/maintenance.php';
    }

#### Manager `$app` Property

**影響の可能性： 低い**

The previously deprecated `$app` property of the `Illuminate\Support\Manager` class has been removed. If you were relying on this property, you should use the `$container` property instead.

#### The `elixir` Helper

**影響の可能性： 低い**

The previously deprecated `elixir` helper has been removed. Applications still using this method are encouraged to upgrade to [Laravel Mix](https://github.com/JeffreyWay/laravel-mix).

### メール

#### The `sendNow` Method

**影響の可能性： 低い**

The previously deprecated `sendNow` method has been removed. Instead, please use the `send` method.

### Pagination

<a name="pagination-defaults"></a>
#### Pagination Defaults

**影響の可能性： 高い**

The paginator now uses the [Tailwind CSS framework](https://tailwindcss.com) for its default styling. In order to keep using Bootstrap, you should add the following method call to the `boot` method of your application's `AppServiceProvider`:

    use Illuminate\Pagination\Paginator;

    Paginator::useBootstrap();

### Queue

<a name="queue-retry-after-method"></a>
#### The `retryAfter` Method

**影響の可能性： 高い**

For consistency with other features of Laravel, the `retryAfter` method and `retryAfter` property of queued jobs, mailers, notifications, and listeners has been renamed to `backoff`. You should update the name of this method / property in the relevant classes in your application.

<a name="queue-timeout-at-property"></a>
#### The `timeoutAt` Property

**影響の可能性： 高い**

The `timeoutAt` property of queued jobs, notifications, and listeners has been renamed to `retryUntil`. You should update the name of this property in the relevant classes in your application.

<a name="failed-jobs-table-batch-support"></a>
#### Failed Jobs Table Batch Support

**影響の可能性： 状況による**

If you plan to use the [job batching](/docs/{{version}}/queues#job-batching) features of Laravel 8.x, your `failed_jobs` database table will need to be updated. First, a new `uuid` column should be added to your table:

    use Illuminate\Database\Schema\Blueprint;
    use Illuminate\Support\Facades\Schema;

    Schema::table('failed_jobs', function (Blueprint $table) {
        $table->string('uuid')->after('id')->unique();
    });

Next, the `failed.driver` configuration option within your `queue` configuration file should be updated to `database-uuids`.

### Scheduling

#### The `cron-expression` Library

**影響の可能性： 低い**

Laravel's dependency on `dragonmantank/cron-expression` has been updated from `2.x` to `3.x`. This should not cause any breaking change in your application unless you are interacting with the `cron-expression` library directly. If you are interacting with this library directly, please review its [change log](https://github.com/dragonmantank/cron-expression/blob/master/CHANGELOG.md).

### セッション

#### The `Session` Contract

**影響の可能性： 低い**

The `Illuminate\Contracts\Session\Session` contract has received a new `pull` method. If you are implementing this contract manually, you should update your implementation accordingly:

    /**
      * Get the value of a given key and then forget it.
      *
      * @param  string  $key
      * @param  mixed  $default
      * @return mixed
      */
     public function pull($key, $default = null);

### テスト

<a name="assert-exact-json-method"></a>
#### The `assertExactJson` Method

**影響の可能性： 中程度**

The `assertExactJson` method now requires numeric keys of compared arrays to match and be in the same order. If you would like to compare JSON against an array without requiring numerically keyed arrays to have the same order, you may use the `assertSimilarJson` method instead.

### バリデーション

### Database Rule Connections

**影響の可能性： 低い**

The `unique` and `exists` rules will now respect the specified connection name (accessed via the model's `getConnectionName` method) of Eloquent models when performing queries.

<a name="miscellaneous"></a>
### その他

We also encourage you to view the changes in the `laravel/laravel` [GitHub repository](https://github.com/laravel/laravel). While many of these changes are not required, you may wish to keep these files in sync with your application. Some of these changes will be covered in this upgrade guide, but others, such as changes to configuration files or comments, will not be. You can easily view the changes with the [GitHub comparison tool](https://github.com/laravel/laravel/compare/7.x...master) and choose which updates are important to you.