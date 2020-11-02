# 管理 Schema

我們已經看了一些可用於與資料交互的新方法。 當然，在一些特定情況下，可能需要直接與資料庫進行交互。

## 資料庫適配器

XF2 中預設的資料庫適配器是基於 MySQL 和 PHP 的 mysqli 擴展。 在任何 XF 類中都可以使用以下方法訪問設定的資料庫適配器：

```php
$db = \XF::db();
```

適配器有許多可用的方法，這些方法將執行一個 SQL 查詢，然後將結果格式化為一個陣列。 例如，要訪問一個用戶記錄：

```php
$db = \XF::db();
$user = $db->fetchRow('SELECT * FROM xf_user WHERE user_id = ?', 1);
```

`$user` 變數現在將包含一個陣列，其中包含查詢結果集中第一行的所有值。 要想從該查詢中獲得單一的值，例如用戶名，可以執行以下操作：

```php
$username = $user['username'];
```

!!! warning
    直接編寫並傳遞給資料庫適配器的資料庫查詢並非自動 "安全"。 如果用戶的輸入沒有經過清除，並且沒有經過準備就傳入查詢，它們就會帶來 SQL 隱碼攻擊漏洞的風險。 正確的方法是使用準備好的語句，就像上面的例子一樣。 在查詢本身中使用 `?` 佔位符來表示參數。 這些佔位符經過適當的轉義後，會被下一個參數中的值替換。 如果你需要使用的參數不止一個，那應該以陣列的形式傳遞到 fetch 類型方法中。 如有必要，你可以直接使用 `$db->quote($value)` 對值進行轉義或引用。

    您可以 [在這裡](http://php.net/manual/en/mysqli.quickstart.prepared-statements.php) 找到更多關於準備好的語句資訊。

也可以從一條記錄中查詢一個值。 例如：

```php
$db = \XF::db();
$username = $db->fetchOne('SELECT username FROM xf_user WHERE user_id = ?', 1);
```

如果你有一個需要返回多條記錄的查詢，你可以使用 `fetchAll`：

```php
$db = \XF::db();
$users = $db->fetchAll('SELECT * FROM xf_user LIMIT 10');
```

或者 `fetchAllKeyed`：

```php
$db = \XF::db();
$users = $db->fetchAllKeyed('SELECT * FROM xf_user LIMIT 10', 'user_id');
```

這兩種方法都將返回一個代表每個用戶記錄的陣列。 `fetchAll` 和 `fetchAllKeyed` 方法之間的區別是，返回的陣列的 key 值不同。 使用 `fetchAll` 方法時，陣列將用連續的數字整數進行 key 鎖定。 用 `fetchAllKeyed` 方法，陣列將用第二個參數中命名的欄位名進行 key 鎖定。

!!! note
    如果你使用 `fetchAllKeyed`，請注意第二個參數是要對陣列進行 key 鎖定的欄位，但 **第三個** 參數是你傳遞 param 值以匹配 `?` 佔位符的地方。

還有一些其他獲取類型的方法，包括 `fetchAllColumn`，用於從所有返回的 rows 中抓取一個特定 column 的值的陣列：

```php
$db = \XF::db();
$usernames = $db->fetchAllColumn('SELECT username FROM xf_user LIMIT 10');
```

上面的例子將返回一個從結果查詢中找到的 10 個用戶名的陣列。

最後，你可能並不想要或不需要返回任何資料，在這種情況下，你可以只做一個普通查詢：

```php
$db = \XF::db();
$db->query('DELETE FROM xf_user WHERE user_id = ?', 1);
```

## 綱要管理

XF2 包含了一種全新的管理資料庫模式的方式，它採用物件導向的方式來執行某些表的操作。 讓我們先看看傳統的改變，使用資料庫適配器，就像我們上面的那樣。

```php
$db = \XF::db();
$db->query("
    ALTER TABLE xf_some_existing_table
    ADD COLUMN new_column INT(10) UNSIGNED NOT NULL DEFAULT 0,
    MODIFY COLUMN some_existing_column varchar(250) NOT NULL DEFAULT ''
");
```

我們也來看看一個典型的建立資料表查詢：

```php
$db = \XF::db();
$sm = $db->getSchemaManager();

$defaultTableConfig = $sm->getTableConfigSql();

$db->query("
    CREATE TABLE xf_some_table (
        some_id INT(10) UNSIGNED NOT NULL AUTO_INCREMENT,
        some_name VARCHAR(50) NOT NULL,
        PRIMARY KEY (user_id)
    ) {$defaultTableConfig}
");
```

在 XF2 中，另一種和首選的方法是使用新的 `SchemaManager` 物件。 讓我們來看看這兩個由模式管理器執行的查詢，首先是 alter：

```php
$sm = \XF::db()->getSchemaManager();
$sm->alterTable('xf_some_existing_table', function(\XF\Db\Schema\Alter $table)
{
    $table->addColumn('new_column', 'int')->setDefault(0);
    $table->changeColumn('some_existing_column')->length(250);
});
```

還有資料表的建立：

```php
$sm = \XF::db()->getSchemaManager();
$sm->createTable('xf_some_table', function(\XF\Db\Schema\Create $table)
{
    $table->addColumn('some_id', 'int')->autoIncrement();
    $table->addColumn('some_name', 'varchar', 50);
});
```

!!! warning
    當您更改現有的 XenForo 資料表或建立自己的資料表時，**必須** 指定一個預設值，否則在查詢資料表時將會遇到問題。

這兩個例子產生的查詢結果和上面更直接的例子完全一樣。 不過你可能會注意到，有些東西是（故意）遺漏的。 例如，沒有一個例子為 `int` 欄位指定一個長度。 這只是因為省略了這一點，MySQL 會給它提供一個預設值，對於無符號整數來說是 10。說到這裡，我們也沒有指定 `some_id` column 是無符號的。 在 XF 內使用無符號整數是目前最常見的使用案例，所以會自動新增。 如果你真正需要支持負整數的能力，你可以用 `->unsigned(false)` 方法反過來。 另一個遺漏是沒有為所有的東西定義 `NOT NULL`。 同樣，這是自動應用的，但你可以用 `->nullable(true)` 來扭轉這種情況。

從 alter 的例子中可能看不清楚，但是當改變現有的欄位時，現有的欄位定義會自動保留。 這意味著，你不必指定整個 column 的定義，包括所有沒有實際改變的部分，你可以只指定你要改變的部分。

關於 Primary Key，還有一些其他的自動推斷。 如果你願意的話，你可以明確地定義 Primary Key（或任何其他類型的 Key ），但通常自動遞增的欄位通常會成為你的資料表的 Primary Key。 所以在建立資料表的例子中，`some_id` 欄位被自動分配為該資料表的 Primary Key。

最後，對於建立資料表的方法，我們可以為指定的儲存引擎自動新增正確的資料表設定（預設為 `InnoDB`，但可以很容易地改變為其他引擎類型）。