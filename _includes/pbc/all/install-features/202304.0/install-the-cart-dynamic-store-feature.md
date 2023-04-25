
This document describes how to integrate the Cart + Dynamic Store feature into a Spryker project.

## Install feature core

{% info_block errorBox %}

The following feature integration guide expects the basic feature to be in place. The current feature integration guide only adds the *Cart Dynamic store functionality*.

{% endinfo_block %}

### Prerequisites

To start feature integration, overview and install the necessary features:

| NAME | VERSION |
| --- | --- |
| Spryker Core | {{page.version}} |
| Cart | {{page.version}} |

### 1) Set up behavior


Register the following plugins:

| PLUGIN                                      | SPECIFICATION                                                                                                                             | PREREQUISITES      | NAMESPACE                                                |
|---------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------|--------------------|----------------------------------------------------------|
| QuoteSyncDatabaseStrategyReaderPlugin       | Sets retrieved quote from Persistence in session storage in case of persistent strategy and `QuoteTransfer.id` is empty.                  | None               | Spryker\Zed\PriceCartConnector\Communication\Plugin      |



**src/Pyz/Client/Quote/QuoteDependencyProvider.php**

```php
<?php

namespace Pyz\Client\Quote;

use Spryker\Client\PersistentCart\Plugin\Quote\QuoteSyncDatabaseStrategyReaderPlugin;
use Spryker\Client\Quote\QuoteDependencyProvider as SprykerQuoteDependencyProvider;

class QuoteDependencyProvider extends SprykerQuoteDependencyProvider
{
    /**
     * @return array<\Spryker\Client\QuoteExtension\Dependency\Plugin\DatabaseStrategyReaderPluginInterface>
     */
    protected function getDatabaseStrategyReaderPlugins(): array
    {
        return [
            new QuoteSyncDatabaseStrategyReaderPlugin(),
        ];
    }
}
```

{% info_block warningBox "Verification" %}

Make sure Zed request is made in case of persistent strategy and `QuoteTransfer.id` is empty. Also, make sure that a quote is retrieved from Persistence using the provided customer in case of persistent strategy and `QuoteTransfer.id` is empty. 
Finally, make sure that the retrieved quote from Persistence is set in session storage in case of persistent strategy and `QuoteTransfer.id` is empty.

{% endinfo_block %}
