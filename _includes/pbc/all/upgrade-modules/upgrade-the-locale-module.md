This document describes how to upgrade the Locale module.

## Upgrading from version 3.* to version 4.0.0

In this new 4.0.0 version of the `Locale` module, we have added support configuration locale for each store in the database.
With the new 4.0.0 version of the `Locale` module, we have also added the `spy_locale_store` database table to persist stores-locales in Zed.
We also added the column `fk_locale` into  `spy_store`, to allow saving a default locale per store. 

You can find more details about the changes on the [Locale module](https://github.com/spryker/locale/releases) release page.

*Estimated migration time: 5 minutes*

To upgrade to the new version of the module, do the following:

1. Upgrade the `Locale` module to the new version:

```bash
composer require spryker/locale:"^4.0.0" --update-with-dependencies
```

2. Run `vendor/bin/console transfer:generate` to update the transfer objects.

3. Run `vendor/bin/console propel:install` to apply the database changes.


***


{% info_block errorBox %}

This migration guide is a part of the [Silex migration effort](/docs/scos/dev/migration-concepts/silex-replacement/silex-replacement.html).

{% endinfo_block %}

To upgrade the module, do the following:

1. Update the module using Composer:

```bash
composer update spryker/locale
```

2. Add new plugins to dependency providers:

**Zed integration**

```php
<?php

namespace Pyz\Zed\Application;

use Spryker\Zed\Application\ApplicationDependencyProvider as SprykerApplicationDependencyProvider;
use Spryker\Zed\Locale\Communication\Plugin\Application\LocaleApplicationPlugin;

class ApplicationDependencyProvider extends SprykerApplicationDependencyProvider
{
    ...

    /**
     * @return \Spryker\Shared\ApplicationExtension\Dependency\Plugin\ApplicationPluginInterface[]
     */
    protected function getApplicationPlugins(): array
    {
        return [
            ...
            new LocaleApplicationPlugin(),
            ...
        ];
    }

    ...
}
```

**Yves integration**

```php
<?php

namespace Pyz\Yves\ShopApplication;

use Spryker\Yves\ShopApplication\ShopApplicationDependencyProvider as SprykerShopApplicationDependencyProvider;
use Spryker\Yves\Locale\Plugin\Application\LocaleApplicationPlugin;

class ShopApplicationDependencyProvider extends SprykerShopApplicationDependencyProvider
{
    ...

    /**
     * @return \Spryker\Shared\ApplicationExtension\Dependency\Plugin\ApplicationPluginInterface[]
     */
    protected function getApplicationPlugins(): array
    {
        return [
            ...
            new LocaleApplicationPlugin(),
            ...
        ];
    }

    ...
}
```

3. Enable additional plugins:

**EventDispatcher Zed integration**

```php
<?php

namespace Pyz\Zed\EventDispatcher;

use Spryker\Zed\EventDispatcher\EventDispatcherDependencyProvider as SprykerEventDispatcherDependencyProvider;
use Spryker\Zed\Locale\Communication\Plugin\EventDispatcher\LocaleEventDispatcherPlugin;

class EventDispatcherDependencyProvider extends SprykerEventDispatcherDependencyProvider
{
    /**
     * @return \Spryker\Shared\EventDispatcherExtension\Dependency\Plugin\EventDispatcherPluginInterface[]
     */
    protected function getEventDispatcherPlugins(): array
    {
        return [
            ...
            new LocaleEventDispatcherPlugin(),
            ...
        ];
    }
}
```

**EventDispatcher Yves integration**

```php
<?php

namespace Pyz\Yves\EventDispatcher;

use Spryker\Yves\EventDispatcher\EventDispatcherDependencyProvider as SprykerEventDispatcherDependencyProvider;
use Spryker\Yves\Locale\Plugin\EventDispatcher\LocaleEventDispatcherPlugin;

class EventDispatcherDependencyProvider extends SprykerEventDispatcherDependencyProvider
{
    /**
     * @return \Spryker\Shared\EventDispatcherExtension\Dependency\Plugin\EventDispatcherPluginInterface[]
     */
    protected function getEventDispatcherPlugins(): array
    {
        return [
            ...
            new LocaleEventDispatcherPlugin(),
            ...
        ];
    }
}
```
