
This document describes how to integrate the Prices + Dynamic store feature into a Spryker project.

## Install feature core

### Prerequisites

To start feature integration, overview and install the necessary features:

| NAME | VERSION |
| --- | --- |
| Spryker Core | {{page.version}} |
| Prices | {{page.version}} |

### 1) Install the required modules using Composer

```bash
composer require spryker-feature/prices:"{{page.version}}" --update-with-dependencies
```

{% info_block warningBox "Verification" %}

Make sure that the following modules have been installed:

| MODULE | EXPECTED DIRECTORY |
| --- | --- |
| Price | vendor/spryker/price |
| PriceDataFeed | vendor/spryker/price-data-feed |
| Currency | vendor/spryker/currency |
| CurrencyDataImport | vendor/spryker/currency-data-import |

{% endinfo_block %}


### 2) Set up the database schema and transfer objects

1. Adjust the schema definition so entity changes trigger events:

**src/Pyz/Zed/Currency/Persistence/Propel/Schema/spy_currency.schema.xml**

```xml

<?xml version="1.0"?>
<database xmlns="spryker:schema-01" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" name="zed"
          xsi:schemaLocation="spryker:schema-01 https://static.spryker.com/schema-01.xsd"
          namespace="Orm\Zed\Currency\Persistence" package="src.Orm.Zed.Currency.Persistence">

    <table name="spy_currency_store" idMethod="native">
        <behavior name="event">
            <parameter name="spy_currency_store_all" column="*"/>
        </behavior>
    </table>

</database>

```

2. Run the following commands to apply database changes and generate entity and transfer changes:

```bash
console propel:install
console transfer:generate
```

{% info_block warningBox "Verification" %}

Make sure that the following changes have been applied by checking your database:

| DATABASE ENTITY                       | TYPE   | EVENT   |
|---------------------------------------|--------|---------|
| spy_store.fk_currency                 | column | added   |
| spy_currency_store                    | table  | added   |

{% endinfo_block %}


{% info_block warningBox "Verification" %}

Make sure that the following changes have been applied in transfer objects:

| TRANSFER | TYPE | EVENT | PATH |
| --- | --- | --- | --- |
| CurrencyCriteria | class | created | src/Generated/Shared/Transfer/CurrencyCriteriaTransfer  |
| Store.defaultCurrencyIsoCode     | property | added | src/Generated/Shared/Transfer/CustomerTransfer  |

{% endinfo_block %}

### 3) Configure export to Redis

1.  Set up publisher plugins:

| PLUGIN | SPECIFICATION | PRERQUISITES | NAMESPACE |
| --- | --- | --- | --- |
| CurrencyStoreWritePublisherPlugin | Publishes currency store data to storage table. | None | Spryker\Zed\StoreStorage\Communication\Plugin\Publisher\CurrencyStore |


**src/Pyz/Zed/Publisher/PublisherDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Publisher;

use Spryker\Zed\Publisher\PublisherDependencyProvider as SprykerPublisherDependencyProvider;
use Spryker\Zed\StoreStorage\Communication\Plugin\Publisher\CurrencyStore\CurrencyStoreWritePublisherPlugin;

class PublisherDependencyProvider extends SprykerPublisherDependencyProvider
{
    protected function getPublisherPlugins(): array
    {
        return array_merge(
            // ...
            $this->getStoreStoragePlugins(),
        );
    }
    
    /**
     * @return array<\Spryker\Zed\PublisherExtension\Dependency\Plugin\PublisherPluginInterface>
     */
    protected function getStoreStoragePlugins(): array
    {
        return [
            new CurrencyStoreWritePublisherPlugin(),
        ];
    }
}
```


### 4) Import data

Import locale, store and country data:

1.  Prepare your data according to your requirements using our demo data:

Example for DE store currency-store configurations:
**data/import/common/DE/currency_store.csv**

```csv
currency_code,store_name,is_default
EUR,DE,1
CHF,DE,0
```

| Column | REQUIRED | Data Type | Data Example | Data Explanation |
| --- | --- | --- | --- | --- |
| currency_code | mandatory | string | EUR | Define currency code. |
|store_name |mandatory |string | DE | Define store name. |
|is_default |mandatory |bool | 1 | Defines if the currency is default. |

__Note: Only one currency can be default per store.__

{% info_block warningBox “Verification” %}

Make sure that:

1.  The .csv files have an empty line in the end.
2.  For each `currency_code` entry in csv files, there is a respective `code` entry in the table `spy_currency` in the database.
 
{% endinfo_block %}

2. Update the following import action files with the following action:
    * `data/import/common/commerce_setup_import_config_{SPRYKEREGIONR_STORE}.yml`
    * `data/import/local/full\_{REGION\_STORE}.yml`
    * `data/import/production/full\_{SPRYKER\_STORE}.yml`

```yaml
data_import:
    - data_entity: currency-store
      source: data/import/common/{REGION}/currency_store.csv
```

3. Register the following plugins to enable data import:
 

| PLUGIN | SPECIFICATION | PREREQUISITES | NAMESPACE |
| --- | --- | --- | --- |
| CurrencyStoreDataImportPlugin | Imports currency store relations. | None | \Spryker\Zed\CurrencyDataImport\Communication\Plugin\DataImport |

**src/Pyz/Zed/DataImport/DataImportDependencyProvider.php**

```php
namespace Pyz\Zed\DataImport;

use Spryker\Zed\Kernel\Container;
use Spryker\Zed\DataImport\DataImportDependencyProvider as SprykerDataImportDependencyProvider;
use Spryker\Zed\CurrencyDataImport\Communication\Plugin\DataImport\CurrencyStoreDataImportPlugin;

class DataImportDependencyProvider extends SprykerDataImportDependencyProvider
{
    protected function getDataImporterPlugins(): array
    {
        return [
            new CurrencyStoreDataImportPlugin(),
        ];     
    }
}
```

4. Enable behaviors by registering the console commands:

**src/Pyz/Zed/Console/ConsoleDependencyProvider.php**

```php
<?php
namespace Pyz\Zed\Console;

use Spryker\Zed\Console\ConsoleDependencyProvider as SprykerConsoleDependencyProvider;
use Spryker\Zed\DataImport\Communication\Console\DataImportConsole;
use Spryker\Zed\Locale\Communication\Plugin\Application\ConsoleLocaleApplicationPlugin;
use Spryker\Zed\CurrencyDataImport\CurrencyDataImportConfig;

/**
 * @SuppressWarnings(PHPMD.ExcessiveMethodLength)
 * @method \Pyz\Zed\Console\ConsoleConfig getConfig()
 */
class ConsoleDependencyProvider extends SprykerConsoleDependencyProvider
{
    /**
     * @var string
     */
    protected const COMMAND_SEPARATOR = ':';

    /**
     * @param \Spryker\Zed\Kernel\Container $container
     *
     * @return array<\Symfony\Component\Console\Command\Command>
     */
    protected function getConsoleCommands(Container $container): array
    {
        return [
            new DataImportConsole(DataImportConsole::DEFAULT_NAME . static::COMMAND_SEPARATOR . CurrencyDataImportConfig::IMPORT_TYPE_CURRENCY_STORE),
        ];
    }

}
```


5. Import data:

```bash
vendor/bin/console data:import:currency-store    
```

{% info_block warningBox "Verification" %}

Make sure that warehouse and warehouse address data have been added to the `spy_currency_store` table.

{% endinfo_block %}


### 5) Set up behavior

Register the following plugins:

| PLUGIN | SPECIFICATION | PREREQUISITES | NAMESPACE |
| --- | --- | --- | --- |
| CurrencyBackendGatewayApplicationPlugin |Provides currency service. | None | Spryker\Zed\Currency\Communication\Plugin\Application | 
| DefaultCurrencyStorePreCreateValidationPlugin | Validates default currency before store is created. | None | Spryker\Zed\Currency\Communication\Plugin\Store |
| DefaultCurrencyStorePreUpdateValidationPlugin | Validates default currency before store is updated. | None | Spryker\Zed\Currency\Communication\Plugin\Store |
| CurrencyStorePostUpdatePlugin | Update currency store data after store is updated. | None | Spryker\Zed\Currency\Communication\Plugin\Store |
| CurrencyStoreCollectionExpanderPlugin | Expands currency store collection. | None | Spryker\Zed\Currency\Communication\Plugin\Store |
| CurrencyStoreFormExpanderPlugin | Adds currency selection fields to the Store form. | None | Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui |
| CurrencyStoreFormViewExpanderPlugin | Adds rendered currency tabs and tables as variables in template. | None | Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui |
| CurrencyStoreFormTabExpanderPlugin | Expands Store form with Currencies tab. | None | Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui |
| AssignedCurrenciesStoreViewExpanderPlugin | Returns table with assigned currencies. | None | Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui |
| CurrencyStoreTableExpanderPlugin | Expands table data rows of store table with currency codes. | None | Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui |



**src/Pyz/Zed/Application/ApplicationDependencyProvider.php**

```php 
<?php

namespace Pyz\Zed\Application;

use Spryker\Zed\Application\ApplicationDependencyProvider as SprykerApplicationDependencyProvider;
use Spryker\Zed\Currency\Communication\Plugin\Application\CurrencyBackendGatewayApplicationPlugin;


class ApplicationDependencyProvider extends SprykerApplicationDependencyProvider
{
    /**
     * @return array<\Spryker\Shared\ApplicationExtension\Dependency\Plugin\ApplicationPluginInterface>
     */
    protected function getBackendGatewayApplicationPlugins(): array
    {
        return [
            new CurrencyBackendGatewayApplicationPlugin(),
        ];
    }
}
```

{% info_block warningBox "Verification" %}

Make sure service container has `currency` service.

{% endinfo_block %}

**src/Pyz/Zed/Store/StoreDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\Store;

use Spryker\Zed\Currency\Communication\Plugin\Store\CurrencyStoreCollectionExpanderPlugin;
use Spryker\Zed\Currency\Communication\Plugin\Store\CurrencyStorePostCreatePlugin;
use Spryker\Zed\Currency\Communication\Plugin\Store\CurrencyStorePostUpdatePlugin;
use Spryker\Zed\Currency\Communication\Plugin\Store\DefaultCurrencyStorePreCreateValidationPlugin;
use Spryker\Zed\Currency\Communication\Plugin\Store\DefaultCurrencyStorePreUpdateValidationPlugin;
use Spryker\Zed\Store\StoreDependencyProvider as SprykerStoreDependencyProvider;

class StoreDependencyProvider extends SprykerStoreDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\StoreExtension\Dependency\Plugin\StorePreCreateValidationPluginInterface>
     */
    protected function getStorePreCreateValidationPlugins(): array
    {
        return [ 
            new DefaultCurrencyStorePreCreateValidationPlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\StoreExtension\Dependency\Plugin\StorePreUpdateValidationPluginInterface>
     */
    protected function getStorePreUpdateValidationPlugins(): array
    {
        return [ 
            new DefaultCurrencyStorePreUpdateValidationPlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\StoreExtension\Dependency\Plugin\StorePostCreatePluginInterface>
     */
    protected function getStorePostCreatePlugins(): array
    {
        return [
            new CurrencyStorePostCreatePlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\StoreExtension\Dependency\Plugin\StorePostUpdatePluginInterface>
     */
    protected function getStorePostUpdatePlugins(): array
    {
        return [
            new CurrencyStorePostUpdatePlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\StoreExtension\Dependency\Plugin\StoreCollectionExpanderPluginInterface>
     */
    protected function getStoreCollectionExpanderPlugins(): array
    {
        return [
            new CurrencyStoreCollectionExpanderPlugin(),
        ];
    }
}
```

{% info_block warningBox "Verification" %}

Steps: 
- Make sure that you get an error message if you try to create a store with a default currency that is not assigned to the default store.
- Make sure that you get an error message if you try to update a store with a default currency that is not assigned to the default store.
- Make sure that `spy_currency_store` table has been updated with the default currency of the default store for created store or updated store.
- Make sure expanded store transfers have currency codes.
 
{% endinfo_block %}


**src/Pyz/Zed/StoreGui/StoreGuiDependencyProvider.php**

```php
<?php

namespace Pyz\Zed\StoreGui;

use Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui\AssignedCurrenciesStoreViewExpanderPlugin;
use Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui\CurrencyStoreFormExpanderPlugin;
use Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui\CurrencyStoreFormTabExpanderPlugin;
use Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui\CurrencyStoreFormViewExpanderPlugin;
use Spryker\Zed\CurrencyGui\Communication\Plugin\StoreGui\CurrencyStoreTableExpanderPlugin;
use Spryker\Zed\StoreGui\StoreGuiDependencyProvider as SprykerStoreGuiDependencyProvider;

class StoreGuiDependencyProvider extends SprykerStoreGuiDependencyProvider
{
    /**
     * @return array<\Spryker\Zed\StoreGuiExtension\Dependency\Plugin\StoreFormExpanderPluginInterface>
     */
    protected function getStoreFormExpanderPlugins(): array
    {
        return [
            new CurrencyStoreFormExpanderPlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\StoreGuiExtension\Dependency\Plugin\StoreFormViewExpanderPluginInterface>
     */
    protected function getStoreFormViewExpanderPlugins(): array
    {
        return [
            new CurrencyStoreFormViewExpanderPlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\StoreGuiExtension\Dependency\Plugin\StoreFormTabExpanderPluginInterface>
     */
    protected function getStoreFormTabsExpanderPlugins(): array
    {
        return [
            new CurrencyStoreFormTabExpanderPlugin(),
        ];
    }

    /**
     * @return array<\Spryker\Zed\StoreGuiExtension\Dependency\Plugin\StoreViewExpanderPluginInterface>
     */
    protected function getStoreViewExpanderPlugins(): array
    {
        return [
            new AssignedCurrenciesStoreViewExpanderPlugin(),
 
        ];
    }

    /**
     * @return array<\Spryker\Zed\StoreGuiExtension\Dependency\Plugin\StoreTableExpanderPluginInterface>
     */
    protected function getStoreTableExpanderPlugins(): array
    {
        return [
            new CurrencyStoreTableExpanderPlugin(),
        ];
    }
}

```
{% info_block warningBox "Verification" %}

Steps:
- Make sure that you can see the currency selection fields on the Store form.
- Make sure that you can see the currency tabs and tables on the Store form.
- Make sure that you can see the Currencies tab on the Store form.
- Make sure that you can see the table with assigned currencies on the Store form.

{% endinfo_block %}