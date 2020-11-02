# 附加元件結構

在以前的 XF 版本中，很少有關於附加元件開發的標準和約定。 在 XF 2.0 的功能中，我們已經做了很多修改。 讓我們來看看其中的一些變化：

## 附加元件 ID 和附加元件路徑

每個安裝的附加元件都必須有一個唯一的 ID，這個 ID 決定了附加元件應該在檔案系統中的哪個位置儲存其檔案。附加元件的 ID 有兩種可能的格式。

第一個 "簡單" 類型應該是一個單字，不包含任何特殊字元。例如，`Demo`。

簡單的附加元件 ID 必須遵守以下規則：

* 必須只包含 a-z 或 A-Z。
* 可以包含 0-9，但不能放在 ID 開頭。
* 不能包含任何特殊字元，例如斜線、破折號或下劃線。

第二個包含一個供應商前綴，所以如果你在一個特定的品牌或公司下發布附加元件，附加元件 ID 可以表明這一點。例如，`SomeVendor/Demo`。

供應商類型的附加 ID 應遵守以下規則：

* 必須只包含 a-z 或 A-Z
* 可包含一個 `/` 字元，但不能在開頭或結尾處
* 可以包含 0-9，但不在附加元件 ID 的任何一個部分的開頭

一旦您決定了附加元件的 ID，我們就知道這個附加元件的檔案將儲存在哪裡。 所有 XF 2.0 附加元件都儲存在 `src/addons` 目錄的子目錄中。

如果你有一個簡單的附加元件 ID，例如 `Demo`，你的附加元件檔案將儲存在以下位置：
 `src/addons/Demo`。

如果你有一個基於廠商的附加元件 ID，例如 `SomeVendor/Demo`，檔案將儲存在以下位置：
`src/addons/SomeVendor/Demo`。

您選擇的附加元件 ID 也將成為您的類別命名空間前綴（更多資訊請參見 [命名空間](/general-concepts/#_3)）。

## 建議版本字串格式

XF 本身使用 MAJOR.MINOR.PATCH 原則（例如 2.0.0 代表第一個穩定的 XF2 版本）來進行版本編號，我們建議對您自己的附加元件的版本編號採取類似的方法。 基本上，在版本號中增加

* MAJOR 版本，當你做了重大的功能改變，特別是破壞了向後相容性的改變
* MINOR 版本，當你新增功能時，最好是以向後相容的方式新增
* PATCH 版本，當你進行向後相容的 Bug 修復時使用

## 建議版本 ID 格式

附加元件的版本 ID 是基本的整數，用於內部版本比較。 它讓我們可以更容易地檢測到一個版本比另一個版本舊。 您的附加元件每個版本都應該將版本號 ID 至少增加 1，但我們內部對 XF 本身使用的慣例，對附加元件也有潛在的作用。 我們的版本 ID 的格式為 `aabbccde`。

* `aa` 代表 MAJOR 版本
* `bb` 代表 MINOR 版本
* `cc` 代表 PATCH 版本
* `d` 代表狀態，例如 `1` 代表 alpha 版本，`3` 代表 beta 版本，`5` 代表候選版本，`7` 代表穩定版本
* `e` 代表版本 Level 狀態

例如，一個版本號是 1.7.3 候選版本 4 它的附加元件 ID 為 "1070354"。 最終穩定版 XF2 的ID為 `2000070`。 XF 的 1.5.0 Beta 3 版本的 ID 為 `1050033`。 穩定版 99.99.99 會有一個ID為 `99999970` ... 也許你應該慢一點 😉。

## 常見的附加元件檔案和目錄
 
在附加元件的目錄中，有一些檔案和目錄有特殊的用途和意義。

### addon.json 檔案

`addon.json` 是一個包含一些資訊的檔案，這些資訊是幫助 XF 2.0 識別附加元件並在 Admin CP 中顯示其資訊所必需的。 最起碼，你的 `addon.json` 檔案應該看起來像這樣：

```json
{
    "title": "My Add-on by Some Company",
    "version_string": "2.0.0",
    "version_id": 2000070,
    "dev": "Some Company"
}
```

建立附加元件時，會自動為您建立一個基本檔案。

包含一個有效的 `addon.json` 檔案是你的附加元件被識別的必要條件，你可以隨時 [驗證你的 addon.json 檔案](/development-tools/#addonjson_1)。

#### 屬性

Property  | Description
------------- | -------------
`legacy_addon_id`  | 當從 XenForo 1 升級至 XenForo 2 時，用來開啟自動處理附加元件 ID 更改。
`title` | 附加元件的標題。 這將顯示在管理面板中。
`description` | 附加元件的描述。這將顯示在管理面板中。
`version_id` | XenForo 用來追蹤附加元件更新的內部 ID。 每次發佈時必須遞增。
`version_string` | 人類可以讀懂的附加元件版本。 這將在管理面板中顯示，而不是 `version_id` 屬性。
`dev` | 附加元件開發者的名字。這將顯示在管理面板中。
`dev_url` | 如果設定了，開發者的名字將以超連結的形式顯示在管理面板中，並以 (href) 此為目標。
`faq_url` | 如果設定了，一個 FAQ 的超連結將顯示在管理面板中，並以 (href) 此為目標。
`support_url` | 如果設定了，支援幫助的超連結將顯示在管理面板中，並以 (href) 此為目標。
`extra_urls` | 這允許您顯示與附加元件相關的其他內容連結（可能是錯誤報告連結，手冊 - 隨您喜歡）。 <br>一個 JSON 物件的陣列，其中 key 是連結文字，value 是連結目標 (href)。
`require` | XenForo 允許安裝附加元件時需要滿足的一系列要求。 更多資訊請參見 ['require 屬性'](#require)。
`icon` | 資源的圖示。 這可以是一個 Font Awesome 圖示名稱（例如 `fa-shopping-bag`，或一個圖片檔案的路徑）。

##### require 屬性

如果環境不支援或不滿足要求，require 屬性是阻止附加元件安裝或升級的標準方法。 
你可以用它來要求先安裝其他附加元件，要求某些 PHP 擴展必須存在或啟用，和/或 強制執行一個最低的 PHP 版本。

這是一個示例片段：

```json
...
  "require": {
      "XF": [2000010, "XenForo 2.0.0+"],
      "php": ["5.4.0", "PHP 5.4.0+"],
      "php-ext/json": ["*", "JSON extension"]
  }
...
```

每個 require，都是一個命名陣列。

- 陣列的名稱是產品 ID (如 `XF` 或 `php` )。 
- 第一個陣列元素是產品的版本 (如 `2000010` 或 `5.4.0` )。 你可以使用 `*` 來表示產品的任何版本。 
- 第二個陣列元素是該 require 人類可以閱讀的文字，這是在訊息中使用的內容（例如 `XenForo 2.0.0+` 或 `PHP 5.4.0+`）。

下面是受支援的產品 ID 的概述：

產品 / Require 名稱  | 參考...| 值|
------------- | -------------|-------------|
`XF`  | XenForo 的安裝版本。 | XenForo 版本 ID，例如 `200010`。 <br>你可以通過檢查 `/src/XF.php` 檔案頂部的 `$versionId` 定義或者列印 `/XF::$versionId` 的值來獲取目前 XenForo 的版本。
`php`  | PHP 版本。 | PHP 版本，例如 `5.4.0`。 <br>建議你盡可能降低這個值；更新一個 PHP 版本可能是一個相當複雜的工作 - 特別是當其他附加元件與新的 PHP 版本衝突時。
`php-ext / (extension name)`| 一個 PHP 擴展 - 其中 `(extension name)` 是擴展的名稱。 | PHP 擴展版本。 <br>這是用 PHP 的 `version_compare` 函數來檢查的，所以它甚至適用於官方完整 PHP 格式的版本字串，比如 `7.1.19-1+ubuntu16.04.1+deb.sury.org+1`。
`(any addon ID)`  | 任何 XenForo 附加元件，如 `Demo/Addon`。如果您不確定某個附加元件的 ID，請查看它的 `addon.json` 檔案。 | 附加元件的版本 ID。 <br>您可以參考 [建議版本 ID 格式](#id_1) 了解更多資訊。

### hashes.json 檔案

`hashes.json` 是為檔案健檢系統新增支援的新方法，而且最好的部分是 - 它是自動生成的！

作為構建過程的一部分（稍後會有更多說明），我們將對您所有附加元件的檔案進行快速清點，並將計算出的檔案內容雜湊碼寫入其中。


### Setup.php 檔案

`Setup.php` 是您安裝、升級或解除安裝附加元件時需要執行的任何程式碼的新主頁。

[下面](#setup) 我們將詳細介紹如何建立一個 Setup 類。

### \_data 目錄

`_data` 目錄是儲存您的附加元件主要資料的地方。 每個附加元件的資料類型都會有自己的 XML 檔案（而不是所有類型的單個檔案）。這些檔案的雜湊值被包含在 `hashes.json` 中，因此我們可以在允許安裝附加元件之前確保附加元件擁有完整和一致的資料。

### \_output 目錄

`_output` 目錄不是成功安裝附加元件所必需的，而且在發佈附加元件時也不應該包括在內。 這個目錄純粹是為了開發目的，只有在啟用開發模式時才會使用（參見 [啟用開發模式](/development-tools/#_3)）。

每項附加元件資料都儲存在一個單獨的檔案中。 大多數情況下，它們被儲存成 JSON 檔案，但對於短語，它們被儲存為 TXT 檔案，對於模板，它們被儲存為 HTML/CSS/LESS 檔案。 所有的模板類型都可以在檔案系統中直接編輯，對這些檔案的修改會在載入時自動寫回資料庫。

## Setup 類

要為您的附加元件建立一個 Setup 類，您需要做的就是在您附加元件目錄的根目錄下建立一個名為 `Setup.php` 的檔案。

Setup 類應該繼承 `/XF/AddOn/AbstractSetup`，它至少需要實作 `install()`、 `upgrade()` 和 `uninstall()` 方法。 下面是一個簡單的附加元件 Setup 類的樣子。

```php
<?php

namespace Demo;

class Setup extends \XF\AddOn\AbstractSetup
{
	public function install(array $stepParams = [])
	{
		$this->schemaManager()->createTable('xf_demo', function(\XF\Db\Schema\Create $table)
		{
			$table->addColumn('demo_id', 'int');
		});
	}
	
	public function upgrade(array $stepParams = [])
	{
		if ($this->addOn->version_id < 1000170)
		{
			$this->schemaManager()->alterTable('xf_demo', function(\XF\Db\Schema\Alter $table)
			{
				$table->addColumn('foo', 'varchar', 10)->setDefault('');
			});
		}
	}
	
	public function uninstall(array $stepParams = [])
	{
		$this->schemaManager()->dropTable('xf_demo');
	}
}
```
Setup 類也支援在不同的步驟中執行每個操作。 為了實現這種行為，您的 Setup 類可以使用 `StepRunnerInstallTrait` 、 `StepRunnerUpgradeTrait` 和/或 `StepRunnerUninstallTrait` [特性](http://php.net/manual/en/language.oop5.traits.php)。 這些都會自動實作所需的方法，你只需要新增相關的步驟，例如 `installStep1()` 、 `upgrade1000170Step1()` 、 `upgrade1000170Step2()` 和 `uninstallStep1()`，其中升級方法中的 `1000170` 等... 是附加元件的版本 ID (參見[建議版本 ID 格式](#id_1))。
