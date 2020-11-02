# 開發工具

XF2 為開發者提供了大量的內建工具，您可以使用這些工具來加快附加元件的開發，下面我們將介紹其中的一些。

## 除錯模式

除錯模式可以在你的 `config.php` 中啟用，這將允許你訪問 Admin CP 中的某些開發工具（如建立路由，權限，管理導覽等），它還將在每個頁面的底部啟用一個控制台輸出，詳細說明頁面處理的時間，有多少查詢被執行來渲染頁面和使用了多少記憶體。 懸停時，會有一個包含目前 Controller、 Action 和模板名稱等資訊的工具提示。 你也可以點選時間輸出，這將讓你詳細了解到底執行了哪些查詢，以及導致該查詢被執行的堆疊追蹤。

你可以通過在 `config.php` 中新增以下內容來啟用除錯模式：

```php
$config['debug'] = true;
```

## 啟用開發模式

開發模式是一種特殊的模式，在您的 `config.php` 檔案中啟用，它將觸發 XF 自動將您的開發檔案寫入 `_output` 目錄。 這個模式需要在檔案系統模板編輯時啟用。 由於開發模式會將檔案寫入你的檔案系統，確保你有適當的檔案權限是很重要的。 這可能會根據環境的不同而有所不同，但典型的配置是確保你正在使用的任何附加元件的 `_output` 目錄的 chmod 設定為`0777`。 例如，如果你正在使用一個 ID 為 `Demo` 的附加元件，它的開發輸出將被寫入到 `src/addons/Demo/_output`，因此該目錄需要是完全可寫的。

啟用開發模式，也會自動啟用 [除錯模式](#_2)。

要啟用開發模式，請在你的 `config.php` 檔案中新增以下行：

```php
$config['development']['enabled'] = true;
$config['development']['defaultAddOn'] = 'SomeCompany/MyAddOn';
```

`defaultAddOn` 值是可選的，但當建立新內容時，添加該值會在 XF Admin CP 中自動填充指定的附加元件，該附加元件將與附加元件相關聯。

除了上述，你可能會發現有必要新增一些額外的配置，特別是當你使用多個 XF 安裝時。

```php
$config['enableMail'] = false;
```

這將關閉所有從您的論壇發送的郵件。 如果您使用的是真實用戶和真實郵件地址的實時資料副本，這一點尤為重要（儘管我們建議不要這樣做！）。

除了直接關閉郵件之外, 您可以考慮使用一個服務, 例如 [MailTrap.io](https://mailtrap.io)。 這將為您提供一個免費的郵箱，可以接收所有從您的論壇發送的郵件，這對於測試您的新附加元件可能發送的任何電子郵件非常有用。

```php
$config['cookie']['prefix'] = 'anything_';
```

如果您在同一域名上使用兩個或更多的 XF 安裝，您可能會遇到 Cookie 被覆蓋的問題，這是由安裝共享相同的 Cookie 前綴引起的。 因此，建議您確保為您設定的每個 XF 安裝更改 Cookie 前綴。 如果不這樣做，您將會遇到一些問題，例如，當登錄到另一個 XF 安裝時，會被註銷。

## 開發指令

XF 2.0 提供了一些通用的開發和附加元件的 CLI 指令，這些指令的目的是幫助您更有效地開發，甚至可能使一些常見的過程自動化/腳本化。

在本節中，我們將介紹一些常用的工具，並解釋它們的作用。

## 附加元件特定指令

### 新建一個附加元件

!!! terminal
    *$* php cmd.php xf-addon:create

`xf-addon:create` 指令是如何初始設定和建立一個新的附加元件。 一旦它執行，你只需要回答一些基本問題：

* 輸入此附加元件的 ID
* 輸入標題
* 輸入版本 ID（如 1000010 ）。
* 輸入版本字串（如 1.0.0 Alpha）。

然後你會得到一個選項來建立附加元件並寫出它的 addon.json 檔案，並詢問你一些關於是否要新增 Setup.php 檔案的問題。

### 匯出 \_data .XML 檔案

!!! terminal
    *$* php cmd.php xf-addon:export _[addon_id]_

這條指令將用於匯出所有附加元件的資料到 `_data` 目錄下的 XML 檔案。 它從目前資料庫中匯出資料（而不是從開發輸出檔案中匯出）。

### 升級您的附加元件版本

!!! terminal
    *$* php cmd.php xf-addon:bump-version _[addon_id]_ --version-id 1020370 --version-string 1.2.3

!!! note
	如果你的版本字串包含空格，你需要用引號將其包圍。

該指令獲取您的附加元件的附加元件 ID、新版本 ID 和新版本字串。 這樣您就可以輕鬆完成升級附加元件的版本，而無需自己執行升級和重建。 上面的選項是可選的，如果沒有提供這些選項，系統會提示您輸入。 如果您只指定版本 ID，如果它符合我們[推薦的版本ID格式](/add-on-structure/#id_1)，我們將嘗試並自動從中推斷出正確的版本字串。 一旦指令完成，它將自動更新 `addon.json` 檔案和資料庫，並提供正確的版本詳細資訊。

### 同步你的 addon.json 到資料庫

!!! terminal
    *$* php cmd.php xf-addon:sync-json _[addon_id]_

有時你可能更喜歡直接編輯 JSON 檔案中的某些細節，這可能是版本，或新的圖示，或更改標題或描述。 以這種方式更改 JSON 可能會導致附加元件系統認為有待定的更改或附加元件是可升級的。 如果您尚未匯出目前資料，則重建或升級可能是一種破壞性的操作。 因此，建議執行此指令作為匯入該資料的方式，而不影響您的現有資料。

### 驗證您的 addon.json 檔案

!!! terminal
    *$* php cmd.php xf-addon:validate-json _[addon_id]_

如果你想檢查你的 JSON 檔案是否包含正確的內容和正確的格式，你現在可以驗證它。 驗證器將檢查內容是否可以解碼，是否包含所有正確的必填欄位（如標題和版本 ID ），還檢查是否存在可選的 Key（如描述和圖示）。 如果有任何 Key 丟失，我們將為您提供修復問題的服務。 我們還檢查 JSON 檔案中是否有任何意外的欄位。 這些可能是故意的，也可能是錯別字。 您可以手動執行該命令，或者在構建發行版時自動執行該指令。

### 執行特定的設定步驟

有時，無需進行解除安裝和重新安裝過程即可檢查 Setup class 步驟是否正常運行，這是很有幫助的。

有三條指令可以幫助解決這個問題，這些命令只適用於使用預設的 `StepRunner` 特性建立的 Setup class。

#### 執行安裝步驟

!!! terminal
    *$* php cmd.php xf-addon:install-step _[addon_id]_ _[step]_

#### 執行升級步驟

!!! terminal
    *$* php cmd.php xf-addon:upgrade-step _[addon_id]_ _[version]_ _[step]_

#### 執行解除安裝步驟

!!! terminal
    *$* php cmd.php xf-addon:uninstall-step _[addon_id]_ _[step]_

## 構建附加元件發行版

一旦所有的艱苦工作都完成了，很遺憾，在真正發佈之前還要經歷一些其他的過程。 即使是將所有的檔案收集到正確的地方，並手動建立 ZIP 檔案的過程也是很耗時的，而且容易出現錯誤。 我們可以自動處理這些問題，包括用一個簡單的指令產生 `hash.json` 檔案。

!!! terminal
    *$* php cmd.php xf-addon:build-release _[addon_id]_

當你執行這個命令時，它會先執行 `xf-addon:export` 指令，然後將所有的檔案一起收集到一個臨時的 `_build` 目錄中，並將它們寫入一個 ZIP 檔案。 完成後的 ZIP 檔案還將包含 `hashes.json` 檔案。 一旦 ZIP 檔案被建立，它將被儲存到你的 `_releases` 目錄中，並命名為 `<ADDON ID>-<VERSION STRING>.zip`。

### 自定義構建過程

除了建立發佈的 ZIP 之外，您可能還希望在 ZIP 中包含其他的檔案，您希望執行其他更高級的構建過程，如最小化或連線 JS 或執行某些 shell 指令。 所有這些都可以在你的 `build.json` 檔案中處理。 這是一個典型的 `build.json` 檔案。

```json
{
	"additional_files": [
		"js/demo/portal"
	],
	"minify": [
		"js/demo/portal/a.js",
		"js/demo/portal/b.js"
	],
	"rollup": {
		"js/demo/portal/ab-rollup.js": [
			"js/demo/portal/a.min.js",
			"js/demo/portal/b.min.js"
		]
	},
	"exec": [
		"echo '{title} version {version_string} ({version_id}) has been built successfully!' > 'src/addons/Demo/Portal/_build/built.txt'"
	]
}
```

如果你有一些需要在 add-on 目錄之外提供服務的 assets，比如 JavaScript，你可以使用 `build.json` 中的 `additional_files` 陣列告訴構建過程複製檔案或目錄。 在開發過程中，將檔案儲存在 add-on 目錄之外並不總是可行的，所以如果你願意，你可以將檔案儲存在 add-on 的 `_files` 目錄中。 在複製附加檔案時，我們將首先檢查該檔案。

如果您的附加元件附帶了一些 JS 檔案，出於效能考慮，您可能會希望對這些檔案進行最小化。 您可以在您的 `build.json` 中指定您要最小化的檔案。 您可以將這些檔案以陣列的形式列出，也可以直接指定為 `'*'`，這樣就會對 `js` 目錄中的所有檔案進行最小化，只要該路徑中存在 JS 檔案，並將額外的檔案複製到構建中即可。 任何被最小化的檔案都會自動以 `.min.js` 作為後綴，而不是 `.js` ，並且原始檔案仍然在包中。

你可能更喜歡將多個 JS 檔案捆成一個檔案。 如果你這樣做，你可以使用 `rollup` 陣列來定義。 key 是合併後的檔名，陣列中的 item 是 JS 檔案的路徑，這些檔案將被合併成一個檔案。

最後，您可能需要在打包構建和最終定案之前執行某些程序。 這可能是任何東西的組合。 歸根結底，如果它是一個可以從 shell 中執行的指令（包括 PHP 腳本），那麼你可以在這裡指定它。 當然，上面的示例是毫無用處的，但它至少證明了某些保留區是可以使用的。 這些保留區被替換為標量值，你可以從 `XF\AddOn\AddOn` 物件中獲得，一般來說，這個物件是 `addon.json` 檔案中的任何可用值，或者 `AddOn` 實體。

## 開發指令

其實與開發相關的指令有不少，但這裡只介紹最重要的兩個。

要使用這些指令，你必須在你的 `config.php` 檔案中啟用[開發模式](#_3)。

!!! warning
	如果出現資料庫和 `_output` 目錄不同步的情況，以下兩個指令都有可能導致資料丟失。建議使用 VCS (版本控制系統)，如[GitHub](https://github.com) 來減輕這種錯誤的影響。

### 匯入開發輸出

!!! terminal
    *$* php cmd.php xf-dev:import --addon _[addon_id]_

執行此指令將把你的附加元件 `_output` 目錄下的所有開發時輸出的檔案匯入進資料庫裡。

### 匯出開發輸出

!!! terminal
    *$* php cmd.php xf-dev:export --addon _[addon_id]_

這將把目前資料庫中與附加元件相關聯的所有資料匯出到 `_output` 目錄下的檔案中。

## 除錯程式碼

應該可以設定你喜歡的除錯工具（ XDebug，Zend Debugger 等）來與 XF2 一起運作。 雖然，有時候，除錯程式碼可能像快速查看一個變數在給定時間內持有的值（或值類型）一樣簡陋。

### Dump 變數

當然 PHP 有一個內建的工具來處理這個問題，您可能將其稱為 `var_dump()`。 XF 提供了兩個替代方法來處理這個問題：

```php
\XF::dump($var);
\XF::dumpSimple($var);
```

簡單版本主要是將一個變數的值以純文字的形式轉儲出來。 例如，如果你只是用它來轉儲一個陣列的值，你會在頁面頂部看到這樣的輸出：

```plain
array(2) {
  ["user_id"] => int(1)
  ["username"] => string(5) "Admin"
}
```

這實際上是與標準的 var_dump 相同的輸出，但為了可讀性稍作修改，並將其包裹在 `<pre>` 標籤中，以確保在渲染時保留空白。

這個替代方案實際上是 Symfony 項目中的一個名為 VarDumper 的元件。 它輸出 HTML、 CSS 和 JS 來建立一個功能更強、更容易閱讀的輸出。 它允許你摺疊某些部分，對於某些可以輸出大量資料的值，比如物件，它可以自動摺疊這些部分。 