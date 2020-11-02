# 通用概念

以下部分將詳細介紹開發 XenForo 附加元件時將會遇到的一些通用系統和概念。 如果您熟悉 XenForo 1.x 的開發，那麼這些概念中的很多內容您都會覺得很熟悉，不過值得回顧一下，因為有一些優秀的新工具和功能可以幫助您開發附加元件。

## Vendor 元件

XF2 並不像 XF1 那樣由一個特定的框架來驅動，然而，我們使用了某些流行的、經過良好測試的、開源的軟體包來幫助完成特定的任務。 例如，我們使用一個名為 [SwiftMailer](https://github.com/swiftmailer/swiftmailer) 的項目來發送電子郵件，以及一個名為 [Guzzle](https://github.com/guzzle/guzzle) 的項目作為 HTTP 客戶端。 所有的第三方項目都是從 `src/vendor` 目錄下載入的。

目前，附加元件開發者還不能將自己的依賴關係添加到這個位置。

## 整合開發環境 (IDE)

在開始 XF2 開發工作之前，你可能需要花一些時間來評估你將實際建立和編輯 PHP 檔案的應用程式。 這通常被稱為 IDE。 有許多可用的選項，從基本的記事本到像 Sublime Text 這樣可以通過附加元件擴展以獲得更好的 PHP 支持的東西，直到一個合適的 IDE，如 [PhpStorm](https://www.jetbrains.com/phpstorm/)。 在內部，我們使用 PhpStorm 作為我們的首選 IDE。 這是一個高級的商業產品，但也有可能是免費的。 無論哪種方式，沒有人能告訴你最符合您需求的最佳應用程序是哪個，你應該花些時間多使用一些產品（甚至是免費的），並使用這些經驗來找出你的喜好。

## 自動載入器

XF2 使用一個由 Composer 自動產生的自動載入器。 這允許所有的 XF 程式碼、第三方供應商程式碼，以及任何附加元件的開發者程式碼在整個項目中自動包含進去，而不需要手動 `include/require` 你的 class。

所有 XF 附加元件的自動載入根目錄是 `src/addons` 目錄。 這意味著你所有的 class 名都會相對於這個基礎位置。 值得注意的是，XF2 採用了嚴格的 "每個檔案一個類" 的命名模式。 每個檔案應該只包含一個類，而且這個類的名字應該確定該類的 PHP 檔案在檔案系統中的確切位置。

例如，如果你想在一個名為 `src/addons/Demo/Setup.php` 的檔案中建立一個新的 class（其中 `Demo` 是你的 附加元件 ID），那麼這個 class 將被命名為 `Demo\Setup`。 相反，如果你有一個名為 `Demo\Entity\Thing` 的 class，那麼你會知道這個 class 的檔案位於路徑 `src/addons/Demo/Entity/Thing.php` 中。

## 命名空間

在整個 XF 中，我們使用了 [命名空間](http://php.net/manual/en/language.namespaces.rationale.php)，這樣我們可以更簡潔地參考同一名稱空間中的類。建議所有的附加元件也使用名稱空間。 在上面的例子中，我們談到了一個名為 `Demo\Setup` 的類。 使用命名空間，這個類實際上會被簡單地命名為 `Setup`，但命名空間會被設定為 `Demo`。 作為一個更具體的例子，我們在上面也談到了一個名為 `Demo\Entity\Thing` 的類。 讓我們來看看這個類的 PHP 程式碼是什麼樣子的：

```php
<?php

namespace Demo\Entity;

class Thing
{
	
}
```

如果在 `src/addons/Demo/Entity` 目錄下有一個名為 `AnotherThing` 的 class，我們可以在 `Thing` class 中簡單地將這個 class 引用為 `AnotherThing`，因為這個 class 在同一個 `Demo\Entity` 命名空間中。

## 短類名

偶爾，XF 中參考的 class 會被縮短。 例如，如果您希望呼叫 `User` 實體（更多關於實體的內容見下文），那麼您可能會看到被引用的 class 名只是 `XF:User`。 短類名的使用和它們所解析的全類名，完全是依照 context sensitive 的。 因此，在呼叫實體的 context 中，短類名將解析為以下全類名 `XF/Entity/User`。 `XF` 部分表示檔案路徑 (基於附加的 ID )，`Entity` 部分是通過呼叫實體暗示的，`User` 部分表示特定的實體。 同樣當你開始建立自己的 class 時，你也會使用簡短的 class 名來引用自己的 class。 例如，如果你需要為你的 `Demo` 附加元件建立一個新的 `Thing` 實體，那麼你會寫出以下內容。

```php
\XF::em()->create('Demo:Thing');
```

這將解析為 `Demo\Entity\Thing` class。 同樣，如果你想訪問一個 `Thing` 儲存庫，你可以這樣寫。

```php
\XF::repository('Demo:Thing');
```
請注意這些短類名是如何相同的。 儲存庫的呼叫實際上會解析為 `Demo\Repository\Thing`。

## 繼承類

XF2 中大多數的 class 都是可繼承的，這使得開發人員可以繼承和覆蓋核心程式碼，而無需直接編輯它。 如果你熟悉 XF1 的開發，你會對下面的過程有些熟悉：

1. 建立一個監聽器 PHP 檔案
2. 建立一個 class，該 class 最終將繼承原有的 class
3. 編寫一個 function，該 function 與預期的 `load_class` 事件的 callback 簽章相符合，並新增繼承 class 的名稱
4. 在 Admin CP 中新增一個 "程式碼事件監聽器"，指定上述 function 的監聽器 class 和 method 名，並可選地提示繼承的是哪個 class

在 XF2 中，我們去掉了這些事件，而採用了一個名為 "Class extensions" 的特定系統。 這個過程如下。

1. 建立一個 class，該 class 最終將繼承原有的 class
2. 在 Admin CP 中新增一個 "Class extension"，指定你要繼承的 class 的名稱和正在繼承的 class 的名稱。

這顯然減少了一些繼承 class 所需的模板，也提供了一個專門的 UI 來查看和管理這些繼承。 讓我們通過繼承 public `Member` controller 來看看這個過程，並新增一個新的動作來顯示一個簡單的訊息。

首先要做的是建立一個附加元件。 我們之前已經介紹過如何使用 `xf-addon:create` 指令 [在這裡](/development-tools/#_6)。 在這個例子中，我們將假設你建立了一個 ID 和標題為 "Demo" 的附加元件。

現在，您將在以下位置中出現此附加元件的 addon.json 檔案 `src/addons/Demo/addon.json`。

!!! note
	雖然嚴格來說，你可以把繼承 class 放在附加元件目錄下任何你喜歡的地方，但建議把繼承 class 放在一個容易識別的目錄下<br>
	a) class 所屬的附加元件<br>
	b) 被繼承 的 class 類型<br>
	c) 被繼承的 class 名稱<br>
	在下面的例子中，我們要繼承 public 的 XF Member Controller，所以我們將我們的繼承 class 放在下面的路徑中： `src/addons/Demo/XF/Pub/Controller/Member.php`。

在我們將 Class extension 新增到 Admin CP 之前，繼承的 class 需要存在。 所以，請按照下面的說明進行操作：

1. 在 `src/addons/Demo` 內新建一個名為 `XF` 的目錄。
2. 在 `src/addons/Demo/XF` 內新建一個名為 `Pub` 的目錄。
3. 在 `src/addons/Demo/XF/Pub` 內新建一個名為 `Controller` 的目錄。
4. 在 `src/addons/Demo/XF/Pub/Controller` 內新建一個名為 `Member.php` 的檔案。

你的PHP檔案初始內容，應該如下所示：

```php
<?php

namespace Demo\XF\Pub\Controller;

class Member extends XFCP_Member
{
	
}
```

如果你對一般的 PHP class 繼承比較熟悉，但對 XF 不熟悉，上面的例子最初可能會讓你感到困惑。 原因是你可能期望直接繼承 `XF\Pub\Controller\Member` 類目錄，而不是 `XFCP_Member`。 在 XF 中，我們使用 "XenForo Class Proxy" 系統（簡稱 XFCP ）來構建一個 "inheritance chain"，最終允許一個 class 被多個附加元件繼承。 慣例是參考一個虛擬的繼承 class，也就是目前的 class 名 `Member`，並以 `XFCP_` 為前綴。

現在 class 已經建立好了，我們可以在 Admin CP > 開發 > Class extensions > 新增 Class extensions 頁面上創建 Class extension。

你需要做的就是在第一個欄位中輸入基本 class 名（`XF\Pub\Controller\Member`），在第二個欄位中輸入繼承 class 名（你剛剛建立的）（`Demo\XF\Pub\Controller\Member`），然後點選 "儲存" 按鈕。

你的 Class extension 現在應該是被啟用的，但目前沒有做任何事情。 要做一些事情，我們需要通過建立一個與現有 method 同名的 method 來 override 這個 class 中的現有 method，或者建立一個全新的 method。 讓我們來做後者，建立全新的 method。

```php
<?php

namespace Demo\XF\Pub\Controller;

class Member extends XFCP_Member
{
	public function actionHelloWorld()
	{
		return $this->message('Hello world!');
	}
}
```

我們在 [Controller 基礎知識](/controller-basics) 的頁面中會更多的講到 Controller 、 Action 和 Reply，所以現在不用特別擔心了解這些。

現在我們已經新增了一些程式碼到我們的繼承 Controller 中，讓我們來看看它的運作。 簡單地輸入以下網址 (相對於你的論壇URL): `index.php?members/hello-world`。 現在你應該會看到一個 "Hello world!" 的訊息！

如前所述，在一個 class 中也可以覆蓋現有的方法。 例如，如果我們將 `actionHelloWorld()` 改為 `actionIndex()`，那麼你將不再有 "Notable members" 列表，而是顯示 "Hello world!" 訊息！ 事實上，這並不是繼承現有 Controller 動作（或任何 class 方法）的正確方式，但我們會在 [修改 Controller Action Reply（properly）](/controller-basics/#controller-action-properly) 一節中詳細介紹。

## 類型提示

XF 中的很多物件都是通過工廠方法實體化的。 例如，如果我們想實體化一個特定的儲存庫，我們會寫以下內容：

```php
$repo = \XF::repository('Demo:Thing');
```

這是一種非常方便和一致的實體化物件的方式。 我們只要看一眼，就知道會實體化什麼物件。 該方法中產生的程式碼知道如何為我們請求的東西返回正確的物件。

然而，不幸的是，你的 IDE 可能不知道（至少預設情況下是這樣）。 就 IDE 而言，這個方法將返回一個 `XF\Mvc\Entity\Repository` 的物件實體。 這在一定程度上是有用的，但是在特定的 `Demo\Repository\Thing` 物件中可能有很多方法是你的 IDE 不知道的。 這最終意味著，當你試圖在程式碼中使用你的 `$repo` 物件時，你的 IDE 將無法提出建議或自動完成方法名和它所需要的參數。

這就是類型提示的用處，而且大多數 IDE 和一些能夠 "識別 PHP" 的文字編輯器都應該標準地支援這種語法。 我們只需將儲存庫的呼叫修改如下：

```php
/** @var \Demo\Repository\Thing $repo */
$repo = \XF::repository('Demo:Thing');
```

儲存庫呼叫上面的類型提示現在告訴 IDE，`$repo` 涉及到一個由 `Demo\Repository\Thing` class 表示的物件，而不是原來自動推斷的物件。

類型提示在繼承 class 的時候也特別有用。 我們的 Class extension 方法有一個潛在的問題，那就是基本上你的 class 並沒有繼承你要繼承的原 class，而是通過一個實際上並不存在的 class 來代理這個 class，比如 [上面例子](#_5) 中的 `XFCP_Member`。

為了解決這個問題，我們會自動產生一個名為 `extension_hint.php` 的檔案，並將其儲存在你的 `_output` 目錄下。

這就增加了一個 IDE 可以讀取但 PHP 不能讀取的參考，所以 IDE 現在可以理解，當我們在這個繼承 class 的任何方法中使用 `$this` 時，它可以建議並自動完成在 Member Controller 或其父類中可用的方法和屬性。