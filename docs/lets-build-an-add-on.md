# 建立一個附加組件

對於一些人來說，直接進入一個項目是最好的學習方式，我們的目的是在下面的章節中，您將學習如何從頭開始構建一個附加元件。 做好準備；這不是一個簡單的 'Hello world' 類型的演示。 這實際上是一個功能相當齊全的演示插件，涵蓋了 XF2 中的許多概念。

我們要建立的附加元件將允許擁有適當權限的用戶 "精選" 一個主題，並允許該主題在新頁面上顯示。 我們甚至會設定一個流程，自動在特定的論壇中對主題進行精選處理。 我們將為此使用一個新的路由，命名為 `portal`，並最終將其設定為索引頁路由，並設定在瀏覽該頁面時選擇 "Home" 標籤。

## 建立附加元件

在整個附加元件中，我們將使用附加元件 ID 為 `Demo/Portal`。 首先我們需要做的是建立附加元件，為此我們需要打開 命令提示字元 / shell / 終端視窗，將目錄改為你的 XF 安裝根目錄（即 `cmd.php` 所在的位置），然後執行以下指令，並輸入下面顯示的對策：

!!! terminal
    *$* php cmd.php xf-addon:create

    **Enter an ID for this add-on:** Demo/Portal

    **Enter a title:** Demo - Portal

    **Enter a version ID:** This integer will be used for internal variable comparisons.  
    Each release of your addon should increase this number:  1000010

    Version string set to: 1.0.0 Alpha

    **Does this add-on supersede a XenForo 1 add-on? (y/n)** n

    The addon.json file was successfully written out to /var/www/src/addons/Demo/Portal/addon.json

    **Does your add-on need a Setup file? (y/n)** y

    **Does your Setup need to support running multiple steps? (y/n)** y

    The Setup.php file was successfully written out to /var/www/src/addons/Demo/Portal/Setup.php

附加元件現在已經建立，你會發現你在 `src/addons` 目錄下有一個新的目錄，你會在 Admin CP 的 "已安裝 add-ons" 列表中找到該附加元件。

其中一個已經建立的檔案是 `addon.json` 檔案，它目前的樣子是這樣的：

```json
{
    "legacy_addon_id": "",
    "title": "Demo - Portal",
    "description": "",
    "version_id": 1000010,
    "version_string": "1.0.0 Alpha",
    "dev": "",
    "dev_url": "",
    "faq_url": "",
    "support_url": "",
    "extra_urls": [],
    "require": [],
    "icon": ""
}
```

讓我們來填一下這些詳細內容：

```json
{
    "legacy_addon_id": "",
    "title": "Demo - Portal",
    "description": "Add-on which will display featured threads on the forum home page.",
    "version_id": 1000010,
    "version_string": "1.0.0 Alpha",
    "dev": "You!",
    "dev_url": "",
    "faq_url": "",
    "support_url": "",
    "extra_urls": [],
    "require": [],
    "icon": "fa-home"
}
```

現在，我們已經新增了 `description`，開發者的名字 (`dev`)，並指定我們要顯示一個圖示 (`icon`)。 圖示可以是一個路徑（相對於你的 add-on 根目錄），也可以是一個 [Font Awesome 圖示](http://fontawesome.io/icons/) 的名稱，就像我們在這裡做的那樣。

由於我們不是要取代 XenForo 1 附加元件，所以我們可以忽略 `legacy_addon_id`。 關於 `addon.json` 檔案中所有屬性的完整解釋，請參考 [附加元件結構部分](../add-on-structure/#addonjson)。

## 建立 Setup 類

好吧，嚴格來說，這個類已經被建立並寫入了 `Setup.php`，但現在它並沒有真正做什麼。 我們基本上已經有了一個 class 骨架，它看起來像這樣：

```php
<?php

namespace Demo\Portal;

use XF\AddOn\AbstractSetup;
use XF\AddOn\StepRunnerInstallTrait;
use XF\AddOn\StepRunnerUninstallTrait;
use XF\AddOn\StepRunnerUpgradeTrait;

class Setup extends AbstractSetup
{
        use StepRunnerInstallTrait;
        use StepRunnerUpgradeTrait;
        use StepRunnerUninstallTrait;
}
```

我們已經談過一點關於 Setup 類的內容。 我們將把安裝、升級和解除安裝過程分成不同的步驟。

讓我們從匯入一些有用的 Schema 類開始。 如果你想了解更多關於它們的資訊，你可以參考 [管理 Schema 部分](/managing-the-schema)。 
在最後一個 `use` 聲明之後，新增以下幾行：

```php
use XF\Db\Schema\Alter;
use XF\Db\Schema\Create;
```

這裡的 StepRunner 特性將處理所有可用步驟的循環過程，所以我們要做的就是開始建立這些步驟。 我們首先新增一些程式碼，在 `xf_forum` 資料表中建立一個新的 column：

```php
<?php

namespace Demo\Portal;

use XF\AddOn\AbstractSetup;
use XF\AddOn\StepRunnerInstallTrait;
use XF\AddOn\StepRunnerUninstallTrait;
use XF\AddOn\StepRunnerUpgradeTrait;

use XF\Db\Schema\Alter;
use XF\Db\Schema\Create;

class Setup extends \XF\AddOn\AbstractSetup
{
	use StepRunnerInstallTrait;
	use StepRunnerUpgradeTrait;
	use StepRunnerUninstallTrait;

	public function installStep1()
	{
		$this->schemaManager()->alterTable('xf_forum', function(Alter $table)
		{
			$table->addColumn('demo_portal_auto_feature', 'tinyint')->setDefault(0);
		});
	}
}

```

這一 column 被新增到 `xf_forum` 資料表中，這樣我們就可以設定某些論壇在建立主題時自動顯示。 這裡的命名很重要，新增到核心 XF 資料表中的 columns 總是應該有前綴。 這有兩個重要的目的。 第一個是減少了重複 column 名發生衝突的風險，以防 XF 或其他附加元件有理由在將來新增該 column。 第二個目的是，它有助於更容易地識別哪些 columns 屬於哪些附加元件，以防將來出現一些問題。

既然如此，我們不妨在安裝程式中再增加一個步驟。 為了簡潔起見，我們只顯示新程式碼，而不是整個類。 它應該直接放在 `installStep1()` 方法的下面：

```php
public function installStep2()
{
    $this->schemaManager()->alterTable('xf_thread', function(Alter $table)
    {
        $table->addColumn('demo_portal_featured', 'tinyint')->setDefault(0);
    });
}
```

這一步驟與上面的步驟類似，這次將在 `xf_thread` 資料表中新增一個新 column。 我們將使用這一 column 作為一個快取值來快速識別一個主題是否有精選，而不需要針對 `xf_demo_portal_featured_thread` 資料表查找或進行額外的查詢。

說到這裡，我們現在應該把那個資料表加進去了。 這次直接加在 `installStep2()` 下面：

```php
public function installStep3()
{
    $this->schemaManager()->createTable('xf_demo_portal_featured_thread', function(Create $table)
    {
        $table->addColumn('thread_id', 'int');
        $table->addColumn('featured_date', 'int');
        $table->addPrimaryKey('thread_id');
    });
}
```

這一步將建立新的資料表。 這張資料表將用來記錄所有被精選的主題，以及它們被精選的時間。

在命名方面，同樣的原則也適用於此。 一個重要的區別是，所有的資料表都應該另外加上 `xf_` 的前綴。 這樣做的原因是，如果進行了乾淨的 XF 安裝，我們可以刪除所有帶有 `xf_` 前綴的資料表，包括那些由附加元件建立的資料表。

在新增程式碼時，最容易忘記的一件事就是，添加了各種 schema 更改後忘記自己套用的 schema 更改。 你可以使用 CLI 指令執行 安裝/升級 步驟。 在這種情況下，執行以下指令：

!!! terminal
    *$* php cmd.php xf-addon:install-step Demo/Portal 1
    *$* php cmd.php xf-addon:install-step Demo/Portal 2
    *$* php cmd.php xf-addon:install-step Demo/Portal 3

## 繼承論壇實體

到目前為止，我們已經在 `xf_forum` 資料表中新增了一 column，現在是時候繼承論壇 Entity 結構了。 我們需要這樣做，以便 Entity 知道我們的新 column，並且可以通過 Entity 向其讀寫資料。

!!! note
    以下步驟需要啟用 [開發模式](/development-tools/#_3)。 記得在 `config.php` 中設定 `Demo/Portal` 為 `defaultAddOn` 值。

這個過程的第一步是建立一個 "程式碼事件監聽器"。 這可以在開發下的 Admin CP 中，點選 "程式碼事件監聽器" 連結，然後點選 "新增程式碼事件監聽器" 按鈕。

我們需要監聽 `entity_structure` 事件。 我們將使用它來修改預設的論壇 Entity 結構，以新增我們新建立的 `demo_portal_auto_feature` column。

在 "事件提示" 欄位中，我們將輸入要繼承的類名，例如 `XF\Entity\Forum`。 這將確保我們的監聽器只在論壇 Entity 上執行。

對於 "執行 callback" 類輸入 `Demo\Portal\Listener`，對於方法輸入 `forumEntityStructure`。

值得新增一個描述來解釋這個監聽器的用途，因為這將有助於更容易地在程式碼事件監聽器列表中識別監聽器。 "繼承 XF\Entity\Forum 結構" 應該就夠了。 最後，確保 "Demo - Portal" 附加元件被選中。

在點選 "儲存" 之前，我們需要實際建立 Listener 類。 所以在 `src/addons/Demo/Portal` 中建立一個名為 `Listener.php` 的新檔案。 這個檔案的內容最初應該是這樣的。 我們從程式碼事件選擇器下面的檔案中知道這個函數需要的參數。

```php
<?php

namespace Demo\Portal;

use XF\Mvc\Entity\Entity;

class Listener
{
	public static function forumEntityStructure(\XF\Mvc\Entity\Manager $em, \XF\Mvc\Entity\Structure &$structure)
	{

	}
}
```

注意 `namespace` 和 `class` 名稱之間的 `use` 宣告。 我們將不止一次地引用這裡宣告的 class，因此在這裡宣告它確實允許我們用它更短的別名來引用它，在本例中是 `Entity`。

這段程式碼實際上還不會做任何事情，但現在是儲存程式碼事件監聽器的好時機，所以請繼續點選 "儲存" 按鈕。

在我們為新函數添加一些功能程式碼之前，現在也許是個很好的時機來看看開發輸出系統是如何運作的。 檢查一下新增到你 add-on 目錄中的新目錄和檔案。 特別是在 `_output/code_event_listeners` 目錄下有一個新的 JSON 檔案，它應該是這樣的：

```json
{
    "event_id": "entity_structure",
    "execute_order": 10,
    "callback_class": "Demo\\Portal\\Listener",
    "callback_method": "forumEntityStructure",
    "active": true,
    "hint": "XF\\Entity\\Forum",
    "description": "Extends the XF\\Entity\\Forum structure"
}
```

每當監聽器發生變化時，這個檔案會自動更新。

好了，我們再來新增一些程式碼。 回到 `Listener` 類中，在 `forumEntityStructure` 函數中新增以下內容：

```php
$structure->columns['demo_portal_auto_feature'] = ['type' => Entity::BOOL, 'default' => false];
```

論壇 Entity 現在已經知道了我們的新 column，但在開始實作對該 column 進行實際設定值的方法之前，我們還需要先處理幾個步驟。

## 繼承主題實體

同樣，由於我們在 xf_thread 資料表中新增了一個新的 column，我們應該讓主題實體知道這一點。 這與我們上面的作法非常相似。

回到 "新增程式碼事件監聽器"，再次監聽 `entity_structure`。 這次的 "事件提示" 將是 `XF\Entity\Thread`。 我們可以使用與之前相同的 callback 類（`Demo\Portal\Listener`），但這次的方法將命名為 `threadEntityStructure`。 新增類似之前的描述。 在儲存之前，我們應該在 `forumEntityStructure` 函數下面直接新增程式碼：

```php
public static function threadEntityStructure(\XF\Mvc\Entity\Manager $em, \XF\Mvc\Entity\Structure &$structure)
{
	$structure->columns['demo_portal_featured'] = ['type' => Entity::BOOL, 'default' => false];
}
```

這段程式碼與我們在論壇 Entity 結構中新增的程式碼幾乎相同，唯一不同的是 column 名。 但是，我們確實需要新增一些其他的東西。 我們應該建立一個 Entity 關聯，這樣，以後我們需要訪問精選主題 Entity（我們在下一節建立）時，我們就可以很容易地通過 Finder 查詢來實現。 在 `$structure->columns` 行下面新增：

```php
$structure->relations['FeaturedThread'] = [
	'entity' => 'Demo\Portal:FeaturedThread',
	'type' => Entity::TO_ONE,
	'conditions' => 'thread_id',
	'primary' => true
];
```

有關 [關聯](/entities-finders-repositories/#relations) 的更多資訊，請參見關聯。 點選 "儲存" 來儲存監聽器。

## 建立一個新實體

上面在 `installStep3()` 中，我們建立了一個新資料表。 我們需要建立一個 Entity 來與這個資料表互動並建立新的記錄。 因為這是一個全新的 Entity，我們除了在 `src/addons/Demo/Portal/Entity/FeaturedThread.php` 中建立 class 之外，不需要做任何其他的事情，它的架構看起來像這樣：

```php
<?php

namespace Demo\Portal\Entity;

use XF\Mvc\Entity\Structure;

class FeaturedThread extends \XF\Mvc\Entity\Entity
{

}
```

我們需要用它來定義 Entity 結構，它代表我們之前建立的新 `xf_demo_portal_featured_thread` 資料表。 這個 Entity 的結構應該是這樣的：

```php
public static function getStructure(Structure $structure)
{
	$structure->table = 'xf_demo_portal_featured_thread';
	$structure->shortName = 'Demo\Portal:FeaturedThread';
	$structure->primaryKey = 'thread_id';
	$structure->columns = [
		'thread_id' => ['type' => self::UINT, 'required' => true],
		'featured_date' => ['type' => self::UINT, 'default' => time()]
	];
	$structure->getters = [];
	$structure->relations = [
		'Thread' => [
			'entity' => 'XF:Thread',
			'type' => self::TO_ONE,
			'conditions' => 'thread_id',
			'primary' => true
		],
	];

	return $structure;
}
```

根據我們前面寫的 MySQL 建立資料表，columns 的列表大概不需要加以說明。 關聯中包括一個 `Thread` 關聯，它將允許我們從這個 Entity 中獲得相關的主題 Entity 記錄（甚至主題 Entity 關聯）。

## 修改論壇編輯表單

我們現在需要一種方法來修改 `forum_edit` 模板，在那裡新增一個新的 checkbox，最終可以寫回我們現在建立的新 column。 我們將通過建立一個模板修改來實現。 這是在 Admin CP 下的外觀中完成的，然後點選模板修改。 點選 "Admin" 標籤，然後再點選 "新增模板修改" 按鈕。

在 "模板" 欄中，輸入 "forum_edit"。 這就是我們需要修改的模板。

在 "修改 key" 欄位中，輸入 "demo_portal_forum_edit"。 這是一個唯一的 key，用於標識您的模板修改。 這部分最好的習慣是，最起碼要在被修改的模板名稱後面註明附加元件。

"描述" 欄位應該包含一些文字，以幫助您在查看模板修改列表時確定此修改的目的。 就像是 "新增自動精選 checkbox 到 forum_edit 模板" 這樣的內容應該足夠了。

當您在 "模板" 欄位中輸入模板名稱時，您可能會注意到顯示了模板內容的預覽。 我們需要利用這個來確定 checkbox 的偏好位置。 在查看論壇編輯頁面時，你可能會注意到有一系列的 checkbox，這看起來是一個合理的位置。

最簡單的方法是在這個部分放置一個 checkbox，對上面的 checkbox 進行簡單的替換，所以在 "查詢" 欄中新增：

```plain
<xf:option name="allow_posting"
```

並在欄位中替換：

```html
<xf:option name="demo_portal_auto_feature" selected="$forum.demo_portal_auto_feature"
	label="Automatically feature threads in this forum"
	hint="If selected, any new threads posted in this forum will be automatically featured." />
$0
```

我們還不需要擔心建立短語的問題，我們可以稍後再來拾取這些。 注意，名稱屬性必須與我們之前建立的 column 名相匹配，更重要的是，checkbox row 的勾選狀態也是從論壇 Entity 中讀取新增加的 column。

當我們稍後儲存模板修改時，如果查詢欄位的內容與模板的任何部分相匹配，那麼它將被替換欄位的內容所取代。 實際上我們並沒有刪除匹配的內容，因為替換欄位中的 `$0` 是重新插入匹配的文字。

我們可以使用 "測試" 按鈕來檢查替換的運作是否符合預期。 當點擊測試按鈕時，會出現一個覆蓋層帶有修改後的模板。 如果一切順利，一個綠色區域應該高亮顯示我們要添加的新程式碼。

!!! note
    這是一個相當簡單的替換。 對於更進階的匹配，你也可以使用 "正規表達式" 類型。 關於使用正規表達式的詳細解釋超出了本指南的範圍，但網上有很多資源可能會有所幫助。

最後，點選儲存，儲存你的模板修改。 如果一切順利，當你返回模板修改列表時，你會看到日誌摘要顯示
<span style="color: green; font-weight: 700;">1</span> / 0 / <span style="color: red;">0</span>
因此表明修改成功套用一次。 一個更好的指示是進入 Admin CP 中 "論壇" 下的 "節點" 頁面，編輯一個現有的論壇。 現在應該會出現我們新添加的模板修改。

## 繼承論壇的儲存過程

我們有了 column，有了一個 UI 來傳遞 input 到該 column，現在我們必須處理儲存資料到該 column。 我們將通過繼承論壇 Controller 和繼承一個特殊的方法來實現這個目的，當一個節點及其資料被儲存時，該方法將被呼叫。 首先，讓我們建立一個 "Class extension"，它可以在 Admin CP 的 "開發" 條目下找到。 點選 "新增 Class extension"。

在這裡，我們需要指定一個 "父類名稱"，也就是我們要繼承的 class 名稱，在本例中是 `XF\Admin\Controller\Forum`。 並且我們需要指定一個 "繼承 class 名稱"，也就是繼承父類的 class。 輸入 `Demo\Portal\XF\Admin\Controller\Forum`。 我們應該在點選儲存之前建立這個 class。

在 `src/addons/Demo/Portal/XF/Admin/Controller` 中建立一個新檔案，命名為 `Forum.php`。 這可能看起來像一個很長的路徑，但我們建議繼承 class 使用這樣的路徑。 它可以讓你更容易地識別出代表繼承 class 的檔案，因為它們在一個與繼承的 "add-on" ID (本例中為 `XF`) 同名的目錄中。 它還可以清楚地說明到底是哪個 class 被繼承了，因為目錄結構與預設 class 的路徑相同。 目前，檔案的內容應該是這樣的：

```php
<?php

namespace Demo\Portal\XF\Admin\Controller;

class Forum extends XFCP_Forum
{

}
```

更多資訊請參見 [繼承類](/general-concepts/#_5) 和 [類型提示](/general-concepts/#_6)。

點選儲存，儲存 Class extension。 現在我們可以新增一些程式碼了。 我們需要繼承的特定方法是一個名為 `saveTypeData` 的 protected 函數。 當繼承任何類中現有的方法時，檢查原始方法是很重要的，原因有幾個。 第一個原因是我們要確保我們在繼承方法中使用的參數與我們要繼承的方法相匹配。 第二個原因是，我們需要知道這個方法實際上是做什麼的。 例如，這個方法應該返回某個特定類型的東西，還是某個物件？ 這在大多數 Controller action 中肯定是這樣的，我們在 [修改 Controller action 回應（properly）](/controller-basics/#controller-action-properly) 一節中提到過。 然而，雖然這個方法是在一個 Controller 內，但實際上它本身並不是一個 Controller action。 事實上，這個方法是一個 "void" 方法；它不需要返回任何東西。 然而，我們應該始終確保我們在繼承方法中呼叫父類方法，所以我們只需添加新方法本身，而不需要添加新的程式碼：

```php
protected function saveTypeData(FormAction $form, \XF\Entity\Node $node, \XF\Entity\AbstractNode $data)
{
	parent::saveTypeData($form, $node, $data);
}
```

!!! Warning
    此特定方法的參數列表假定我們有一個 `use` 宣告，它將完整的 `\XF\Mvc\FormAction` 類別名為簡單的 `FormAction`。 因此你需要自己新增那個 use 宣告。 在 `namespace` 和 `class` 行之間新增 `use XF\Mvc\FormAction;`。

所以，現在，我們已經繼承了那個方法，而我們的繼承應該被呼叫，但現在它除了呼叫它的父類方法外，沒有做任何事情。 現在我們需要從論壇編輯頁面獲取輸入值，並將其套用到 `$data` 實體（在本例中是論壇實體）。

```php
protected function saveTypeData(FormAction $form, \XF\Entity\Node $node, \XF\Entity\AbstractNode $data)
{
	parent::saveTypeData($form, $node, $data);

	$form->setup(function() use ($data)
	{
		$data->demo_portal_auto_feature = $this->filter('demo_portal_auto_feature', 'bool');
	});
}
```

使用 `FormAction` 物件允許我們在典型的表單提交過程中，具有各種不同的 extension points 到程序中運行。 並不是所有的 Controller action 都可以使用。 例如，它在 Admin CP 中更為普遍，它通常遵循簡單的 CRUD 模型（新增、查詢、修改、刪除）。 XF 中的許多其他程序都發生在 Service 物件內，Service 物件通常有與正在運行的服務相關特定 extension points。 `FormAction` 物件的這種特殊用法與你通常會遇到的情況有些不同。 儲存一個節點是個有些不同的程序，因為除了與節點實體一起運作外，你還會與相關的節點類型一起運作，例如一個論壇 Entity。 不過在這個方法中，我們確實可以訪問表單 action 物件，所以我們應該使用它。 我們在這裡使用它來為程序的 "設定" 階段新增一個特定的行為。 也就是說，當呼叫 `FormAction` 物件的 `run()` 方法時，它將按照特定的順序執行各個階段。 不管這些行為是以哪種順序新增到物件中的，它們仍然會按照 `setup`、`validate`、`apply`、`complete` 的順序執行。

通過上面的程式碼，我們可以將論壇 Entity 中的 `demo_portal_auto_feature` column 設定為我們新增到論壇編輯頁面的 `demo_portal_auto_feature` 輸入存儲的任何值。 現在應該可以測試所有這些工作了。 只需編輯您選擇的論壇並勾選 checkbox。 你應該可以觀察到兩件事。 首先，當你回到編輯那個論壇時，checkbox 現在應該被勾選了。 第二，如果你在 xf_forum 資料表中查看你剛剛編輯的論壇，`demo_portal_auto_feature` 欄位現在應該設定為 1。 請保持這個值，因為我們最終會自動精選該論壇的主題。

## 設置主題自動顯示

我們已經在論壇 Entity 中新增了一個新的 column，這將允許我們在論壇中新建立一個主題時自動進行精選介紹，所以現在是時候新增程式碼來實現這個功能了。

在 XF2 中，我們大量使用 Service 物件。 這些物件通常採用 "setup and go" 類型的方法；你設定你的配置，然後呼叫一個方法來完成動作。 我們使用 Service 物件來設定和完成主題創建，所以這是一個完美的位置來添加我們需要的程式碼。 這一切都要從另一個 Class extension 開始，所以進入 "新增 Class extension" 頁面。

這一次，基類會是 `XF\Service\Thread\Creator`，繼承類則是 `Demo\Portal\XF\Service\Thread\Creator`，和往常一樣，這個新 class 將看起來像下面的程式碼。 在路徑 `src/addons/Demo/Portal/XF/Service/Thread/Creator.php` 中建立該程式碼，然後點選 "儲存" 來建立繼承類。

```php
<?php

namespace Demo\Portal\XF\Service\Thread;

class Creator extends XFCP_Creator
{

}
```

在這裡，我們還將建立另一個繼承類。 基礎類是 `XF\Pub\Controller\Forum`，繼承類是 `Demo\Portal\XF\Pub\Controller\Forum`。 在路徑 `src/addons/Demo/Portal/XF/Pub/Controller/Forum.php` 中建立以下程式碼，並點選 "儲存"：

```php
<?php

namespace Demo\Portal\XF\Pub\Controller;

class Forum extends XFCP_Forum
{

}
```

我們最終將在被繼承的主題作者物件中繼承 `_save()` 方法，這樣我們就可以在建立主題後對其進行精選。 為了配合 "setup and go" 的方針，我們將建立一個方法，它可以用來指示主題是否應該被建立為精選，或者不應該。 為此，我們需要兩樣東西：一個是用來儲存 value 的 class 屬性（預設為 null ）和一個允許設定該屬性的 public 方法。

```php
protected $featureThread;

public function setFeatureThread($featureThread)
{
    $this->featureThread = $featureThread;
}
```

回到我們新繼承的論壇 Controller，我們現在將繼承設定 creator 服務的方法，如果論壇 Entity 有必要的設定值，則選擇加入精選。 請記住，在繼承一個方法之前，我們需要知道它預計會返回什麼（如果有的話），並確保我們呼叫父類方法。 如果父類方法確實返回了些什麼，那麼我們應該在程式碼完成後返回這個方法。 在這種情況下，`setupThreadCreate()` 方法會返回設定好的 creator 服務，所以我們將按照以下方式開始：

```php
protected function setupThreadCreate(\XF\Entity\Forum $forum)
{
    /** @var \Demo\Portal\XF\Service\Thread\Creator $creator */
	$creator = parent::setupThreadCreate($forum);

	return $creator;
}
```

正如預期的那樣，這實際上並沒有做任何事情；繼承的程式碼被呼叫了，但它所做的只是返回父類呼叫所返回的內容。 現在我們應該修改 `$creator` 來設定精選，如果它適用於我們目前正在使用的論壇。

在 `$creator` 行和 `return` 行之間加上：

```php
if ($forum->demo_portal_auto_feature)
{
	$creator->setFeatureThread(true);
}
```

現在我們可以在繼承的 creator 類中新增 `_save()` 方法：

```php
protected function _save()
{
	$thread = parent::_save();

	return $thread;
}
```

為了確保這條主題被精選，在 `$thread` 行和 `return` 行之間，我們只需要新增：

```php
if ($this->featureThread && $thread->discussion_state == 'visible')
{
    /** @var \Demo\Portal\Entity\FeaturedThread $featuredThread */
    $featuredThread = $thread->getRelationOrDefault('FeaturedThread');
    $featuredThread->save();

    $thread->fastUpdate('demo_portal_featured', true);
}
```

因為我們之前在主題 Entity 上建立了 `FeaturedThread` 關聯，所以實際上我們也可以使用這個關聯來建立！ 我們在這裡使用了一個名為 `getRelationOrDefault()` 的方法。 它將查看該關聯是否實際返回一個現有的記錄，如果沒有，它將建立該 Entity 並使用任何預設值（包括主題ID）對其進行設置！ 這意味著我們實際上需要做的只是獲取預設的關聯，並將其儲存下來插入到資料庫中。

此外，我們應該將 `demo_portal_featured` 欄位設定為 true。 因為主題 Entity 已經被儲存了（當原 class 儲存 Entity 時），我們可以使用 `fastUpdate()` 方法來快速修改該欄位。

我們現在需要嘗試這一切，並確保它能正常運作。 前往您之前啟用了 `demo_portal_auto_feature` 選項的論壇，並建立一個新的主題。 現在唯一的方法就是檢查 `xf_demo_portal_featured_thread` 資料表，在那裡我們應該可以看到一條新的記錄。

## 建立 Portal 頁面

在我們完成之前，還有相當多的工作要做，但現在我們已經有了精選主題的能力，如果我們能在某個地方顯示它們，當然會很好，所以讓我們開始建立我們的入口頁面。

要做到這一點，我們需要新建一條 Public 路由。 進入 Admin CP，在 "開發" 下點選 "路由"，然後點選 "添加路由： Public"。 我們暫時先把事情簡單化。 路由的前綴是 "portal"，Section context 是 "home"，Controller 是 "Demo\Portal:Portal"。

現在我們應該在路徑 `src/addons/Demo/Portal/Pub/Controller/Portal.php` 中建立 Controller，基本內容如下：

```php
<?php

namespace Demo\Portal\Pub\Controller;

class Portal extends \XF\Pub\Controller\AbstractController
{

}
```

我們希望當人們訪問 `index.php?portal` 頁面時，我們的 portal 頁面會顯示在他們面前。 這個 URL 沒有 "action" 部分 - 只有我們剛剛建立的路由前綴。 考慮到這一點，我們需要在 `actionIndex()` 方法中新增顯示 portal 頁面的程式碼。 我們在其中需要的基本程式碼是：

```php
public function actionIndex()
{
	$viewParams = [];
	return $this->view('Demo\Portal:View', 'demo_portal_view', $viewParams);
}
```

現在，這還不能完全正常運作，因為我們還沒有建立模板，但這已經足夠了，至少證明了我們的 Route 和 Controller 是相互溝通的。 所以訪問 `index.php?portal` 至少應該顯示 '模板錯誤'。

正如在 [View 回應](/controller-basics/#view) 一節中提到的，第一個參數是一個 View class，但我們並不需要實際建立這個類。 如果有必要，這個 class 可以由其他附加元件繼承，即使它不存在。 第二個參數是模板，我們現在需要在路徑 `src/addons/Demo/Portal/_output/templates/public/demo_portal_view.html` 中建立這個模板。 目前，這個模板應該簡單地包含以下內容。

```html
<xf:title>Portal</xf:title>
```

如果我們現在訪問 portal 頁面，模板錯誤就會消失，雖然我們仍然會有一個看起來相當空白的頁面，但至少現在會有 "Portal" 標題。

現在，是時候開始新增程式碼了，它將顯示精選主題的列表。 第一步是為我們常見的一些基礎 Finder 查詢建立一個 repository。 因此，在路徑 `src/addons/Demo/Portal/Repository/FeaturedThread.php` 中建立一個新檔案，並新增以下程式碼。

```php
<?php

namespace Demo\Portal\Repository;

use XF\Mvc\Entity\Finder;
use XF\Mvc\Entity\Repository;

class FeaturedThread extends Repository
{
	/**
	 * @return Finder
	 */
	public function findFeaturedThreadsForPortalView()
	{
		$visitor = \XF::visitor();

		$finder = $this->finder('Demo\Portal:FeaturedThread');
		$finder
			->setDefaultOrder('featured_date', 'DESC')
			->with('Thread', true)
			->with('Thread.User')
			->with('Thread.Forum', true)
			->with('Thread.Forum.Node.Permissions|' . $visitor->permission_combination_id)
			->with('Thread.FirstPost', true)
			->with('Thread.FirstPost.User')
			->where('Thread.discussion_type', '<>', 'redirect')
			->where('Thread.discussion_state', 'visible');

		return $finder;
	}
}
```

我們在這裡做的是使用 finder 查詢所有的精選主題，按照相反的 `featured_date` 順序，join 到 `xf_thread` 資料表，並從該資料表 join 到主題創建者的 `xf_user` 資料表、`xf_forum` 資料表、`xf_post` 資料表，然後再從那裡 join 到 `xf_user` 資料表，重新創建帖子資料表。 我們通過指定參數 `true` 來確定主題、論壇和第一個帖子必須存在，所以這些將以 `INNER JOIN` 的方式執行，而用戶查詢將以 `LEFT JOIN` 的方式執行。 有些主題和帖子的作者可能不存在（例如，如果它們是由 RSS feed 系統自動發佈的，或由訪客發佈的）。

我們在這裡還有一個特殊的 join，可以在查詢的同時獲取目前訪問者的權限。 這將減少渲染 portal 頁面所需的查詢次數，因為我們將做一些事情（之後），只向有權限查看的用戶顯示精選主題。

這並不能返回這個查詢的結果。 這將返回 Finder 物件本身。 這樣就可以在其他附加元件需要繼承我們的程式碼時，有一個明確的 extension point，同時也允許我們在獲取資料之前做進一步的修改（例如為分頁設定一個 limit/offset，或者設定不同的排序）。

現在讓我們在 Portal Controller 的 `actionIndex()` 方法中使用它。 將現有的這一行 `$viewParams = [];` 改為如下：

```php
/** @var \Demo\Portal\Repository\FeaturedThread $repo */
$repo = $this->repository('Demo\Portal:FeaturedThread');

$finder = $repo->findFeaturedThreadsForPortalView();

$viewParams = [
	'featuredThreads' => $finder->fetch()
];
```

在這個階段，我們不打算擔心修改我們從 repo 中檢索到的基礎 finder。 相反，我們開始實際看到一些結果，並更新 demo_portal_view 模板如下（在 `<xf:title>` 標籤之後）：

```html
<xf:if is="$featuredThreads is not empty">
	<xf:foreach loop="$featuredThreads" value="$featuredThread">
		<xf:macro name="thread_block"
			arg-thread="{$featuredThread.Thread}"
			arg-post="{$featuredThread.Thread.FirstPost}"
			arg-featuredThread="{$featuredThread}"
		/>
	</xf:foreach>
<xf:else />
	<div class="blockMessage">目前還沒有任何帖子被精選。</div>
</xf:if>

<xf:macro name="thread_block" arg-thread="!" arg-post="!" arg-featuredThread="!">
	<xf:css src="message.less" />

	<div class="block">
		<div class="block-container" data-xf-init="lightbox">
			<h4 class="block-header"><a href="{{ link('threads', $thread) }}">{$thread.title}</a></h4>
			<div class="block-body">
				<xf:macro name="message"
					arg-post="{$post}"
					arg-thread="{$thread}"
					arg-featuredThread="{$featuredThread}"
				/>
			</div>
			<div class="block-footer">
				<a href="{{ link('threads', $thread) }}">繼續閱讀...</a>
			</div>
		</div>
	</div>
</xf:macro>

<xf:macro name="message" arg-post="!" arg-thread="!" arg-featuredThread="!">
	<div class="message message--post message--simple">
		<div class="message-inner">
			<div class="message-cell message-cell--main">
				<div class="message-content js-messageContent">
					<div class="message-attribution">
						<div class="contentRow contentRow--alignMiddle">
							<div class="contentRow-figure">
								<xf:avatar user="{$post.User}" size="xxs" defaultname="{$post.username}" href="" />
							</div>
							<div class="contentRow-main contentRow-main--close">
								<ul class="listInline listInline--bullet u-muted">
									<li><xf:username user="{$thread.User}" /></li>
									<li><xf:date time="{$featuredThread.featured_date}" /></li>
									<li><a href="{{ link('forums', $thread.Forum) }}">{$thread.Forum.title}</a></li>
									<li>{{ phrase('replies:') }} {$thread.reply_count|number}</li>
								</ul>
							</div>
						</div>
					</div>
					<div class="message-userContent lbContainer js-lbContainer" data-lb-id="post-{$post.post_id}" data-lb-caption-desc="{{ $post.User ? $post.User.username : $post.username }} &middot; {{ date_time($post.post_date) }}">
                        		    <blockquote class="message-body">{{ bb_code($post.message, 'post', $post.User, { 'attachments': $post.attach_count ? $post.Attachments : [], 'viewAttachments': $thread.canViewAttachments()}) }}</blockquote>
					</div>
				</div>
			</div>
		</div>
	</div>
</xf:macro>
```

現在，我承認，這裡有 **很多** 的內容。 雖然它可能看起來令人生畏，但它主要是以合理的風格來顯示我們的精選主題的標記。 不過有幾件事值得注意。

我們用一個條件開始模板，這個條件是 `<xf:if is="$featuredThreads is not empty">`。 這是為了檢查 finder 返回的物件是否真的包含精選主題記錄。 如果不包含，我們將顯示一個適當的訊息。

如果確實有一些記錄，我們需要循環瀏覽每一條記錄來顯示它。 對於每條記錄，我們呼叫一個 `macro`。 巨集是模板程式碼中可重複使用的部分，它是自行記錄的（你可以看到哪些參數是支援的），並保持自己的作用域，不能被呼叫巨集模板中的參數所污染；這意味著巨集只知道明確傳遞進來的參數和全域的 `$xf` 參數。

主題區塊巨集顯示精選主題的基本區塊，然後呼叫另一個巨集來顯示每個訊息。

## 實作導覽標籤

你可能已經發現，在設定路徑時，我們將 Section context 指定為 "主頁"，當你訪問 portal 頁面時，主頁標籤被選中，或者如果在選項中沒有設定 `homePageUrl`，你可能根本不會看到主頁標籤。 我們希望使用預設的主頁標籤，而不是自己建立一個標籤，這樣可能會有一個重複的標籤。

要做到這一點，我們應該使用程式碼事件監聽器將 URL 改為我們的 portal URL。 在 Admin CP 下的開發中點選 "程式碼事件監聽器"，然後點選 "新增程式碼事件監聽器"。 監聽事件 `home_page_url`，callback class 又會是 `Demo\Portal\Listener`，這次的方法命名為 `homePageUrl`。

這個新方法的程式碼應該相當簡單：

```php
public static function homePageUrl(&$homePageUrl, \XF\Mvc\Router $router)
{
	$homePageUrl = $router->buildLink('canonical:portal');
}
```

最後，我們應該考慮更改我們 portal 頁面的首頁路徑。 進入 Admin CP，在設定下點選選項，然後點選 "基本板塊資訊"。 將 "首頁路徑" 選項改為 `portal/`。

當你在 Admin CP 中時，來看看當你點選 Header 的板塊標題時，會發生什麼。 這應該會帶你到你的首頁。 一切正常，那個首頁現在應該是你的 portal 頁面了！ 除此之外，主頁標籤應該是可見的，並且被選中了。

作為可選步驟，你可以選擇在主頁標籤下新增一些額外的導覽條目。 不過，現在讓我們先繼續往下。

## 手動顯示（或不顯示）主題

所以，我們可以自動精選新主題。 那麼手動顯示現有的主題呢？ 或者是在建立過程中，在不支援自動精選的情況下，手動精選主題？ 這將是一個很好的方法，讓我們目前的 portal 頁面看起來更忙碌。

為了達到這個目的，我們將在一個特定的巨集中新增一個模板修改，這個巨集實際上是在主題回覆、主題編輯和建立主題時使用的。 這將牽涉到繼承編輯器服務，並對處理自動精選的現有程式碼進行修改。

第一步就是新建模板修改。 進到 "新增模板修改"（確保在 "模板修改" 列表中選擇 "Public" 標籤）。 這次我們要修改的模板是 `helper_thread_options`，我們用 `demo_portal_helper_thread_options` 作為 key，你可以寫一個合理的描述。 實際上，我們可以在這裡做一個 "簡單替換"，所以請選中這個單選按鈕，並在 "查詢" 欄位中填上：

```html
<xf:if is="$thread.canLockUnlock()">
```

在 "替換" 欄位中填上：

```html
<xf:if is="($thread.isInsert() AND !$thread.Forum.demo_portal_auto_feature AND $thread.canFeatureUnfeature())
	OR ($thread.isUpdate() && $thread.canFeatureUnfeature())"
>
	<xf:option label="{{ phrase('demo_portal_featured') }}" name="featured" value="1" selected="{$thread.demo_portal_featured}">
		<xf:hint>{{ phrase('demo_portal_featured_hint') }}</xf:hint>
		<xf:afterhtml>
			<xf:hiddenval name="_xfSet[featured]" value="1" />
		</xf:afterhtml>
	</xf:option>
</xf:if>
$0
```

這個條件判斷有點偏長了，但它允許我們在兩個特定的條件下顯示精選 checkbox：
a)如果帖子還沒有建立，並且論壇的自動精選選項被關閉，並且有精選的權限，或者
b)它是一個已經存在的帖子，並且有 精選/非精選 的權限。

一個快速的 "測試" 應該會顯示這個附加的程式碼將被插入到現有的 `<xf:checkboxrow>` 中 "打開" checkbox 上方。 如果這一切看起來都正常，就點選 "儲存"。

我們不得不在這裡直接在修改中使用模板程式碼，因為在已經存在的 input 或 row 標籤中加入一個模板（像我們之前做的那樣）這樣是會無法運作的。 我們現在還需要為標籤和提示建立短語，因為以後將無法檢測到這些了。

在 "外觀" 下進入 "短語" 並點選 "新增短語"。 確保你的附加元件被選中。 第一個短語的 "Title" 將是 "demo_portal_featured"，文字將是簡單的 "精選"。 點選 "儲存並退出"。 再次點選 "新增短語"。 第二個短語的 "標題" 會是 "demo_portal_featured_hint"，文字會是 "精選主題將出現在 Portal 頁面"。

回到我們剛剛新增修改的模板程式碼，你可能已經注意到了一些事情。 我們在主題 Entity 上呼叫了一個方法，`canFeatureUnfeature()`，而這個方法還不存在。 我們最終要用這個來做一個權限檢查，控制用戶是否可以手動對一個主題進行精選操作。

為了新增這個方法，我們需要為 `XF\Entity\Thread` Entity 新建一個 Class extension。 所以，現在就像我們之前做的那樣做。 繼承 class 將是 `Demo\Portal\XF\Entity\Thread`，所以在路徑 `src/addons/Demo/Portal/XF/Entity/Thread.php` 中建立這個 class，內容如下：

```php
<?php

namespace Demo\Portal\XF\Entity;

class Thread extends XFCP_Thread
{
	public function canFeatureUnfeature()
	{
		return true;
	}
}
```

好吧，所以，我們並沒有在這裡做太多有價值的事情。 `canFeatureUnfeature()` 方法現在所做的就是返回 `true`。 以後，我們會實作一些適當的權限，並在這裡新增。

測試一下目前的效果，打開你之前精選的一個主題，在工具選單中選擇 "編輯主題"。 我們應該看到 "設定主題狀態" checkbox row 有我們新增的 "精選" checkbox，而且應該是勾選的，說明這個主題確實是具有精選的。

我們現在可以繼續改變主題編輯器服務來尋找這個值，並相應地進行精選或取消精選。 為此我們需要兩個新的 Class extensions。 回到 "Class extensions" 頁面。 第一個將有一個基類 `XF\Pub\Controller\Thread` 和繼承類 `Demo\Portal\XF\Pub\Controller\Thread`。 第二個將有一個 `XF\Service\Thread\Editor` 的基類和一個 `Demo\Portal\XF\Service\Thread\Editor` 的繼承類。

編輯器服務其實要和我們之前建立的繼承 creator 服務非常相似，所以在相關位置建立。 下面是繼承類的所有程式碼：

```php
<?php

namespace Demo\Portal\XF\Service\Thread;

class Editor extends XFCP_Editor
{
	protected $featureThread;

	public function setFeatureThread($featureThread)
	{
		$this->featureThread = $featureThread;
	}

	protected function _save()
	{
		$thread = parent::_save();

		if ($this->featureThread !== null && $thread->discussion_state == 'visible')
		{
			/** @var \Demo\Portal\Entity\FeaturedThread $featuredThread */
			$featuredThread = $thread->getRelationOrDefault('FeaturedThread', false);

			if ($this->featureThread)
			{
				if (!$featuredThread->exists())
				{
					$featuredThread->save();
					$thread->fastUpdate('demo_portal_featured', true);
				}
			}
			else
			{
				if ($featuredThread->exists())
				{
					$featuredThread->delete();
					$thread->fastUpdate('demo_portal_featured', false);
				}
			}
		}

		return $thread;
	}
}
```

這比 creator 服務中的程式碼要複雜一些。 例如，有可能會出現這樣的情況，一個主題被編輯了，而用戶沒有編輯該主題的權限，因此我們不顯示 checkbox。 在這些情況下，我們不希望自動認為該主題應該是無精選的。 由於類 `$featureThread` 屬性的預設值是 `null`，我們可以使用這個屬性，這樣本質上該屬性有三種狀態。 在這種情況下，`null` 表示 "沒有變化"，`true` 表示我們對該主題進行了精選設定，`false` 表示我們取消了它的精選。

在非精選化的情況下，我們實際上只是通過呼叫 `delete()` 方法來刪除精選主題 Entity。 在這兩種情況下，我們再次使用 `fastUpdate()` 方法更新主題 Entity 中的快取值，以表示目前的精選狀態。

在完成編輯過程之前，我們需要為我們的繼承主題 Controller 新增程式碼，特別是繼承 `setupThreadEdit()` 方法。 整個繼承主題 Controller 的程式碼看起來將會是這樣：

```php
<?php

namespace Demo\Portal\XF\Pub\Controller;

class Thread extends XFCP_Thread
{
	public function setupThreadEdit(\XF\Entity\Thread $thread)
	{
		/** @var \Demo\Portal\XF\Service\Thread\Editor $editor */
		$editor = parent::setupThreadEdit($thread);

		$canFeatureUnfeature = $thread->canFeatureUnfeature();
		if ($canFeatureUnfeature)
		{
			$editor->setFeatureThread($this->filter('featured', 'bool'));
		}

		return $editor;
	}
}
```

這應該足以讓您編輯一個主題，並將其狀態設定為精選（或非精選）。 如果您現在嘗試一下，您應該可以看到主題相應地從您的 portal 頁面出現和消失。

我們需要在主題 Controller 中繼承另一個方法，以處理主題狀態控制也顯示在一些主題回覆表單上的情況。

我們只需要在上面新增的 `setupThreadEdit()` 方法下面新增以下程式碼即可：

```php
public function finalizeThreadReply(\XF\Service\Thread\Replier $replier)
{
	parent::finalizeThreadReply($replier);

	$setOptions = $this->filter('_xfSet', 'array-bool');
	if ($setOptions)
	{
		$thread = $replier->getThread();

		if ($thread->canFeatureUnfeature() && isset($setOptions['featured']))
		{
			$replier->setFeatureThread($this->filter('featured', 'bool'));
		}
	}
}
```

請注意，我們在這個方法中並沒有實際返回任何東西，因為並不期望它返回任何東西。

最後一步，我們需要回到論壇 Controller，稍微修改一下我們現有的程式碼，這樣，如果精選不是自動的，我們可以手動處理。 這應該是相當直接的。 進入您的繼承論壇 Controller，然後將替換這個：

```php
if ($forum->demo_portal_auto_feature)
{
	$creator->setFeatureThread(true);
}
```

改成下面這樣：

```php
if ($forum->demo_portal_auto_feature)
{
	$creator->setFeatureThread(true);
}
else
{
	$setOptions = $this->filter('_xfSet', 'array-bool');
	if ($setOptions)
	{
		$thread = $creator->getThread();

		if ($thread->canFeatureUnfeature() && isset($setOptions['featured']))
		{
			$creator->setFeatureThread($this->filter('featured', 'bool'));
		}
	}
}
```

這和我們已經擁有的基本相同，例如，如果論壇開啟了自動精選，那麼我們只需將該主題設定為精選，否則，我們檢查是否有 checkbox，就像我們對其他情況所做的那樣，將其設定為 checkbox 狀態。

我們現在應該測試建立 3 個主題，以確保其運作正常。 第一個是在開啟了自動精選的論壇中，確保它仍然有效，然後是在沒有開啟自動精選的論壇中，勾選 "精選" checkbox，再勾選它。 假設一切正常，讓我們繼續前進。

## 改進入口網站頁面

雖然，portal 頁面看起來很合理，但我們可以做得更好一些。

首先，我們應該調整我們的程式碼，使我們只顯示 X 個特色主題，我們還應該新增一些頁面導覽。 在這一點上，如果你還沒有的話，可能值得再多精選一些主題，這樣我們就可以實際測試分頁了！

首先，我們需要回到我們的 portal Controller，並在 `actionIndex()` 方法的頂部新增一些程式碼：

```php
$page = $this->filterPage();
$perPage = 5;
```

這裡的第一行是一個特定的 helper 方法，用來獲取目前的頁碼。 第二行是我們每頁要載入多少個項目，這通常來自一個選項，但我們現在將硬寫死為 5。

接下來要做的就是把這一行：

```php
$finder = $repo->findFeaturedThreadsForPortalView();
```

修改成這樣：

```php
$finder = $repo->findFeaturedThreadsForPortalView()
	->limitByPage($page, $perPage);
```

這就改變了我們的查詢，使它能根據我們上面定義的 頁/每頁 值進行 limit。 這將自動計算出目前頁面的正確 limit (`$perPage`) 和 offset (`($page - 1) * $perPage`)。 接下來，我們需要再傳遞一些參數到我們的 view 參數中，所以將下面：

```php
$viewParams = [
	'featuredThreads' => $finder->fetch()
];
```

改變成：

```php
$viewParams = [
	'featuredThreads' => $finder->fetch(),
	'total' => $finder->total(),
	'page' => $page,
	'perPage' => $perPage
];
```

要使用顯示我們的頁面導覽，我們需要知道條目的總數，我們可以通過 `total()` 方法從 finder 中得到，目前的頁碼和我們每頁顯示的數量。

如果你回到 portal，現在你會看到只有 5 個精選主題顯示。 不過，我們現在需要新增頁面導覽。 所以打開 `demo_portal_view` 模板，在 `</xf:foreach>` 標籤後直接新增以下內容：

```html
<xf:pagenav page="{$page}" perpage="{$perPage}" total="{$total}" link="portal" wrapperclass="block" />
```

此時重新載入 portal 頁面，只要你有 5 個以上的精選主題，現在就會在精選主題列表的底部看到頁面導覽。

其他一些可能有助於改善這個頁面的外觀的東西是新增一個側邊欄，或者更準確地說，一個顯示在側邊欄的小組件位置。

小組件位置是在 Admin CP的 "開發" 下新增的。 進入 "小組件位置" 頁面，點選 "新增小組件位置"。 輸入 "位置 ID" 為 `demo_portal_view_sidebar`，"Title" 為 `Demo portal view: Sidebar` 和一個適當的描述。 確保位置已開啟，並選擇了正確的附加元件 ID 後，點選 "儲存"。

要將這個位置新增到模板中，只需在 `<xf:title>` 標籤下面新增以下內容：

```html
<xf:widgetpos id="demo_portal_view_sidebar" position="sidebar" />
```

當然，在沒有新增一些小組件之前，我們還是不會看到側邊欄。 小組件本身是不指派給附加元件的，所以你為這個位置建立的小組件，如果你想在預設情況下發送一些配置好的小組件，將需要新增到 Setup class 中。

為了簡單起見，我們只需要複製目前指派給 `forum_list_sidebar` 位置的小組件（預設情況下）。 所以，我們將把這些新增到 Setup class 中的一個新的 `installStep4()` 方法中：

```php
public function installStep4()
{
	$this->createWidget('demo_portal_view_members_online', 'members_online', [
		'positions' => ['demo_portal_view_sidebar' => 10]
	]);

	$this->createWidget('demo_portal_view_new_posts', 'new_posts', [
		'positions' => ['demo_portal_view_sidebar' => 20]
	]);

	$this->createWidget('demo_portal_view_new_profile_posts', 'new_profile_posts', [
		'positions' => ['demo_portal_view_sidebar' => 30]
	]);

	$this->createWidget('demo_portal_view_forum_statistics', 'forum_statistics', [
		'positions' => ['demo_portal_view_sidebar' => 40]
	]);

	$this->createWidget('demo_portal_view_share_page', 'share_page', [
		'positions' => ['demo_portal_view_sidebar' => 50]
	]);
}
```

當然，別忘了自己執行這個設定步驟：

!!! terminal
    *$* php cmd.php xf-addon:install-step Demo/Portal 4

## 實現權限和優化

現在，我們在 portal 頁面中顯示所有的精選主題，不管訪問者是否有權限查看它們。 這不是很理想；在某些情況下，您可能希望在某些受限制的論壇上發佈主題，並且只讓那些可以正常檢視該論壇的用戶可見。

要做到這一點，我們需要改變我們的程式碼，以便我們 "over-fetch" 我們需要顯示的記錄數量，過濾掉任何看不見的結果，然後將結果集切成我們想要在每頁顯示的實際數量。 這比聽起來要簡單一些。

首先，請進入 Portal Controller，並將這一行：

```php
->limitByPage($page, $perPage);
```

更改成這樣：

```php
->limit($perPage * 3);
```

然後下面再加上：

```php
$featuredThreads = $finder->fetch()
	->filter(function(\Demo\Portal\Entity\FeaturedThread $featuredThread)
	{
		return ($featuredThread->Thread->canView());
	})
	->sliceToPage($page, $perPage);
```

最後將這行：

```php
'featuredThreads' => $finder->fetch(),
```

更改為：

```php
'featuredThreads' => $featuredThreads,
```

你可能在前面的 demo_portal_view 模板中已經發現，我們呈現的每一篇文章也指定其附件：

```plain
'attachments': $post.attach_count ? $post.Attachments : [],
```

現在，這將為每個帖子產生一個額外的查詢。 所以，我們應該嘗試對我們要顯示的所有帖子進行一次查詢，並提前將它們新增到帖子中。 這可能聽起來比實際情況更複雜。 只要在 `->slice(0, $perPage, true);` 行下面新增以下程式碼即可。

```php
$threads = $featuredThreads->pluckNamed('Thread');
$posts = $threads->pluckNamed('FirstPost', 'first_post_id');

/** @var \XF\Repository\Attachment $attachRepo */
$attachRepo = $this->repository('XF:Attachment');
$attachRepo->addAttachmentsToContent($posts, 'post');
```

我們首先使用 `pluckNamed()` 方法獲得一個主題的集合，然後再從主題中獲得一個帖子的集合（以帖子 ID 為 key ）。 一旦我們得到了帖子，我們只需將它們傳遞到附件 repository 的一個特殊方法中就可以了，該方法執行一個單一的查詢，並為每個帖子的附件關聯 "hydrates"。

最後要完成的權限相關事情是建立一個新的權限來控制誰可以手動地對主題進行 精選/取消精選。 要做到這一點，在 Admin CP 的 "開發" 中點選 "權限定義"，然後點選 "新增權限"。 "權限組" 為 "論壇"，"權限 ID" 為  `demoPortalFeature`，"Title" 應該是 `Can feature / unfeature threads`，將 "介面組" 設定為 `Forum moderator permissions`，在選擇了適當的顯示順序並確保選擇了您的附加元件之後，點選 "儲存"。

要真正使用這個權限，我們需要回到我們的繼承主題 Entity，修改 `canFeatureUnfeature()` 方法。 將 `return true;` 替換為：

```php
return \XF::visitor()->hasNodePermission($this->node_id, 'demoPortalFeature');
```

此時，由於權限沒有任何預設值，如果你去編輯任何一個帖子，你應該會發現 "精選" checkbox 不見了。 但是，如果您授予自己該權限，則該c heckbox 將再次出現。 因此，這應該表明了權限的運作是符合預期的!

## 建立一些選項

我們目前每頁只顯示 5 個精選主題，但如果能有更多的顯示選項就更好了。 建立選項很簡單。 雖然不是必須的，但我們首先要建立一個新的選項組，然後向該組新增一個新的選項。

在 Admin CP 的設定然後選項下點選 "新增選項組" 按鈕。 我們將 "用戶組 ID" 稱為 `demoPortal`，並給它一個 "Demo - Portal options" 的標題。 給它一個合適的 "描述" 和 "顯示順序"，然後點選 "儲存"。

現在點選 "新增選項"。 將 "選項 ID" 設定為 `demoPortalFeaturedPerPage`，"Title" 設定為 `Featured threads per page`，編輯格式設定為 `Spin box`，"資料類型" 設定為 `Positive integer`，"預設值" 設定為 "10"。 點擊 "儲存"。

要實現這一點，請回到 portal Controller，並將：

```php
$perPage = 5;
```

更改為：

```php
$perPage = $this->options()->demoPortalFeaturedPerPage;
```

增加另一個選項可能不會有什麼影響。 也許另一個有用的選項是可以將預設的排序順序從 `xf_demo_portal_featured_thread.feartured_date` 改為 `xf_thread.post_date`。 回到 "Demo - Portal options" 組，點選 "新增選項"。

將 "選項 ID" 設定為 `demoPortalDefaultSort`，"Title" 設定為 `Default sort order`，"編輯格式" 設定為 `Radio buttons`。 "格式參數" 設定如下：

 ```plain
 featured_date={{ phrase('demo_portal_featured_date') }}
 post_date={{ phrase('demo_portal_post_date') }}
 ```

最後將 "預設值" 設定為 `featured_date`，點選 "儲存"。

我們需要建立用於單選按鈕標籤的短語，類似於我們之前建立模板修改的短語。

將選項值設定為 "發佈日期"。

嚴格來說，我們可以直接更新我們的 repository 方法來使用新的選項，但是，也許值得看看自定義 finder 方法是如何運作的。 在路徑 `src/addons/Demo/Portal/Finder/FeaturedThread.php` 中建立一個新檔案，內容如下：

```php
<?php

namespace Demo\Portal\Finder;

use XF\Mvc\Entity\Finder;

class FeaturedThread extends Finder
{
	public function applyFeaturedOrder($direction = 'ASC')
	{
		$options = \XF::options();

		if ($options->demoPortalDefaultSort == 'featured_date')
		{
			$this->setDefaultOrder('featured_date', $direction);
		}
		else
		{
			$this->setDefaultOrder('Thread.post_date', $direction);
		}

		return $this;
	}
}
```

正如你所看到的，我們在這裡所做的只是建立了一個相當基本的類，它繼承了 XF `Finder` 物件，並創建一個簡單的方法來查看我們選項的值並套用適當的預設順序。 現在我們可以更新我們的 repository 方法來取代這個方法。

在我們的精選主題 repository 裡面，找到：

```php
->setDefaultOrder('featured_date', 'DESC')
```

並改成：

```php
->applyFeaturedOrder('DESC')
```

最後，更新我們的 portal view 以顯示適當的時間戳大抵如此 - 根據我們的選項值，可以是精選日期或發佈日期。

在 demo_portal_view 模板中將：

```html
<li><xf:date time="{$featuredThread.featured_date}" /></li>
```

更改為：

```html
<li>
	<xf:if is="$xf.options.demoPortalDefaultSort == 'featured_date'">
		<xf:date time="{$featuredThread.featured_date}" />
	<xf:else />
		<xf:date time="{$thread.post_date}" />
	</xf:if>
</li>
```

## 在可見性變更上取消精選

為了解決這個問題，我們需要再次修改主題 Entity，但這次我們將通過 `entity_post_save` 事件來實現。 正如我們在 [實體生命週期](/entities-finders-repositories/#entity) 中提到的，`_postSave()` 方法是在 Entity 被插入或修改後可以執行動作的地方。 最初我們會在一個主題不再可見的時候，取消對該主題的精選。

所以，回到 "新增程式碼事件監聽" 頁面，這次監聽 `entity_post_save` 事件。 這次的事件提示將是 `XF\Entity\Thread`。 對於執行 callback，我們將使用與之前相同的 class (`Dem\Portal\Listener`)，但是我們將在這裡新增一個新的方法，命名為 `threadEntityPostSave`。 現在讓我們新增這個方法，這樣當我們儲存監聽器時，它就會出現：

```php
public static function threadEntityPostSave(\XF\Mvc\Entity\Entity $entity)
{

}
```

點擊 "儲存" 來儲存監聽器。

這個函數的內容相當簡單，我們來看一下：

```php
if ($entity->isUpdate())
{
	$visibilityChange = $entity->isStateChanged('discussion_state', 'visible');
	if ($visibilityChange == 'leave')
	{
		$featuredThread = $entity->FeaturedThread;
		if ($featuredThread)
		{
			$featuredThread->delete();
			$entity->fastUpdate('demo_portal_featured', false);
		}
	}
}
```

我們之前已經取消了主題的精選，但這一次我們要以主題的狀態為條件。 我們可以使用 `isStateChanged` 方法檢測狀態變化。 這將對傳遞進來的 column 名和 value 返回 `enter` 或 `leave`。 例如，如果 `discussion_state` 從 `visible` 變為 `deleted`，那麼在上面的例子中，該方法將返回 `leave`。

一旦我們檢測到自己 "離開" 可見狀態後，就可以直接確定自己有一個精選的主題關聯，刪除它，並更新快取值。

這只涵蓋了主題被暫時刪除或送入批准佇列的情況，我們還需要涵蓋主題被永久刪除的情況。

為此，我們需要另一個監聽器，這次是針對 `entity_post_delete` 事件。 因此，使用相同的 callback class 新增該監聽器，這次的方法名稱為 `threadEntityPostDelete`。 在監聽器 class 中新增以下程式碼：

```php
public static function threadEntityPostDelete(\XF\Mvc\Entity\Entity $entity)
{
	$featuredThread = $entity->FeaturedThread;
	if ($featuredThread)
	{
		$featuredThread->delete();
	}
}
```

點選 "儲存" 以儲存監聽器後，就可以對此進行測試了。 為了測試這個，其實你最好留意一下 xf_demo_portal_featured_thread 資料表，因為到目前為止，程式碼已經不會顯示不可見的主題了，但重要的是不要留下孤兒資料。 一切都很順利，我們已經非常接近完成了......

## 最後一些未交待清楚的事情

說到孤兒資料，每當解除安裝附加元件時，我們都應該對資料庫進行整理。 我們可以在我們之前建立的 Setup class 中完成這項工作。

我們將建立 3 個新方法，對應我們的前 3 個安裝步驟：

```php
public function uninstallStep1()
{
	$this->schemaManager()->alterTable('xf_forum', function(Alter $table)
	{
		$table->dropColumns('demo_portal_auto_feature');
	});
}

public function uninstallStep2()
{
	$this->schemaManager()->alterTable('xf_thread', function(Alter $table)
	{
		$table->dropColumns('demo_portal_featured');
	});
}

public function uninstallStep3()
{
	$this->schemaManager()->dropTable('xf_demo_portal_featured_thread');
}
```

我們不需要建立解除安裝步驟來移除小組件，因為當小組件位置被移除時，它們會被自動移除。 對於我們建立並與附加元件相關聯的任何其他資料也是如此 -- 它們將在解除安裝時自動刪除。

## 構建附加元件

任何附加元件的最後一步，就是發佈它！ 這涉及到從資料庫中提取 XML 檔案（在軟體包中提供並用於安裝），計算每個檔案的雜湊碼，並將其新增到我們的 `hashes.json` 中，並將相關檔案打包成一個 ZIP 檔案。

值得慶幸的是，這可以通過一個 CLI 指令來完成! 只要執行下面的指令就可以了：

!!! terminal
    *$* php cmd.php xf-addon:build-release Demo/Portal

    **Performing add-on export.**

    **Exporting data for Demo - Portal to ../src/addons/Demo/Portal/_data.**

    10/10 [============================] 100%

    **Written successfully.**

    **Building release ZIP.**

    **Writing release ZIP to ../src/addons/Demo/Portal/_releases.**

    **Release written successfully.**

那麼，我們的演示插件就到此結束了！ 如果您想下載這個附加元件的原始碼，請點選這裡：[Demo-Portal-1.0.0 Alpha.zip](/files/Demo-Portal-1.0.0%20Alpha.zip)。
