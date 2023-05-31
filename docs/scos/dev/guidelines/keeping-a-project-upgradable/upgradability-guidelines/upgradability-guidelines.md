---
title: Upgradability guidelines
description: Find solutions to Evaluator violations
template: howto-guide-template
---

The documents in this section will help you resolve the issues related to code evaluation in a way that keeps your code upgradable and up to date with both Spryker's and industry coding standards.

When you get an evaluation error, check the name of the triggered check in the Evaluation output logs. The name is at the beginning of each error log.

Example:
```bash
============================================
DEPENDENCY PROVIDER ADDITIONAL LOGIC CHECKER
============================================

+---+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------+
| # | Message                                                                                | Target                                                                                                                   |
+---+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------+
| 1 | The condition statement if (!static::IS_DEV) {} is forbidden in the DependencyProvider | tests/Acceptance/_data/InvalidProject/src/Pyz/Zed/Console/ConsoleDependencyProvider.php |
+---+----------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------------+


```

In the example, the name is `DEPENDENCY PROVIDER ADDITIONAL LOGIC CHECKER`. To find the documentation for this error, check the name in the table below.

<div class="width-100">


| Check name  | Error message template                                                   | Documentation                                                                                                                                                                          |
| ----------- |--------------------------------------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| DEPENDENCY PROVIDER ADDITIONAL LOGIC CHECKER | The condition statement if {statement} is forbidden in the DependencyProvider | Dependency provider addition logic checker |

</div>