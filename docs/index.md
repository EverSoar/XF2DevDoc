# 入門須知

歡迎使用 XenForo 2！

本文檔旨在幫助您開始 XenForo 2.0 開發。 本文檔的前提條件假設您已熟悉 PHP 和 MySQL。 並非一定要擁有 XenForo 以前版本的經驗，但這將是一個優勢。

在接下來的頁面中，我們將向您簡要介紹如何設置本地伺服器，準備安裝，全新安裝 XenForo 2.0 以及運行 XF2 開發的一些概念。

## 開發人員有什麼新功能？

儘管 XenForo 2.0 為您的論壇及其成員增加了許多改進，但我們仍投入了大量精力來改進 XenForo 的底層框架。 您可以在以下帖子中閱讀更多關於這些更改的更多信息：

 * <a href="https://xenforo.com/community/threads/xenforo-2-0-development-updates-from-xf2demo.139565/post-1205086" target="_blank">
 	XenForo 2 中的開發人員新功能（第1部分）
   </a>
 * <a href="https://xenforo.com/community/threads/xenforo-2-0-development-updates-from-xf2demo.139565/post-1205088" target="_blank">
 	XenForo 2 開發人員的新增功能（第2部分）
   </a>

## 入門須知

XF 開發入門很容易。 您只需要下載檔案，將它們上傳到 Web 伺服器並觸發安裝即可。

如果你還沒有 Web 伺服器，不用擔心，你可以在本地端電腦上設置一個。

## 下載 XF 2.0

要下載 XF 2.0，只需訪問 [客戶區](https://xenforo.com/customers) 並正常登錄即可。 找到正確的許可證，然後單擊 "下載 XenForo" 連結。 選擇您要下載的版本，套件類型並接受許可協議。 最後，單擊下載按鈕以下載檔案。

## XF 2.0 環境要求

從 XF 1.5 開始，運行 XF 2.0 的環境要求已更改。建議的環境要求如下：

* PHP: 5.4.0+ 以上
* MySQL: 5.5+ 以上
* PHP 擴充套件: MySQLi，GD (支持 JPEG)，PCRE，SPL，SimpleXML，DOM，JSON，iconv，ctype，cURL

[下載環境要求測試腳本。](https://xenforo.com/purchase/requirements-zip)

## 設置本地伺服器

設置用於開發的本地 Web 伺服器通常更方便。 通常有兩種方法可以解決此問題：

1. 自己安裝 Apache（或 nginx ），MySQL（或 MariaDB ）和 PHP。
2. 安裝預構建的虛擬機
3. 安裝預構建的 Stack。

自己進行設置比較複雜，但往往可以讓您更好地控制所有設置。

### 預先構建的虛擬機

網路上有多種預構建的虛擬機，這些虛擬機的優勢在於，將所需的所有服務以整齊的方式打包到一個地方即可運行 XenForo，而不必直接在自己的計算機上安裝和維護它們。

一些 XenForo 開發人員使用名為 [Scotch Box](https://box.scotch.io/) 的虛擬機，它包含了運行 XenForo 所需的一切，無需任何配置。 我們有逐步安裝和運行 XenForo 開發伺服器的 [逐步指南](/scotchbox) - 您可以通過運行一些指令，在短短幾分鐘內就使虛擬網路和資料庫伺服器正常運行。

[安裝用於 XenForo 的 Scotch Box 虛擬機](/scotchbox)

### 預構建 Stack

那裡有許多預構建 Stack，它們在功能集，效能和可靠性方面可能會有所不同。Bitnami 維護許多 Stack，包括分別用於 Linux，Mac 和 Windows 的 [LAMP](https://bitnami.com/stack/lamp),
[MAMP](https://bitnami.com/stack/mamp) 和 [WAMP](https://bitnami.com/stack/wamp) Stacks。 它們都包括對 Apache，MySQL 和 PHP 的完全配置安裝，並包括用於管理 MySQL 的 PhPMyAdmin。

## 上傳

要安裝 XF 2.0，您只需要提取從客戶區下載的 ZIP 檔案並上傳其中的一些檔案和目錄。

解壓縮後，您會看到一個名為 `upload` 的目錄。 您需要進入該目錄，然後將檔案和目錄上載到你伺服器的 Web 根目錄。 這通常會在一個名為
`public_html`, `htdocs` 或 `www` 的目錄。


## 創建 src/config.php

如果使用 CLI 安裝 XF 2.0，則需要手動創建 config.php 檔案。 要做到這一點，請在您上載到伺服器的 XF 2.0 檔案中輸入 `src` 目錄。 創建一個名為 config.php 的新檔案，並填入你的 MySQL 伺服器的主機，埠號，用戶名，密碼和資料庫名稱。

!!! note
	確保在 `src` 目錄中創建配置檔案。 該 `library` 目錄僅用於舊版目的。

完成後，它應如下所示：

```php
<?php

$config['db']['host'] = 'localhost';
$config['db']['port'] = '3306';
$config['db']['username'] = 'root';
$config['db']['password'] = 'mypassword';
$config['db']['dbname'] = 'xf2';
```

您現在可以安裝了！

如果您使用的是 MySQL 5.5 及以上版本，並且希望獲得完整的 unicode 支持（例如 emoji 表情），則還應該在安裝前添加以下內容：

```php
$config['fullUnicode'] = true;
```

## 關於檔案權限說明

XenForo 在運行時需要將檔案寫入特定位置。 在正常操作中，這僅限於 `data` 和 `internal_data` 目錄（及其子目錄）。 這些檔案寫入將由附件上傳之類的東西觸發，因此它們通常將由用戶 PHP 像在 Web 伺服器中一樣運行來觸發。 因此，必須確保在這些目錄中設置了權限，以便 Web 伺務器可以對其進行寫入。 您必須先執行此操作，然後才能開始安裝。

當涉及到 CLI 時，這種情況變得更加棘手，因為現在可能有兩個用戶需要能夠寫入檔案。 因此，採取措施避免寫入這些檔案的問題很重要。 這裡有幾個選擇。


1. 在 CLI 和 Web 伺服器使用相同的用戶。 這可以採取以下形式：在運行任何安裝或升級指令（或任何其他將寫入檔案的指令）之前切換到 Web 伺服器用戶。
2. 如果可用，請考慮將 ACL 應用於 `data` 和 `internal_data` 目錄中。 該概念因操作系統和配置而異，但 [此處](http://symfony.com/doc/current/setup/file_permissions.html) 描述的是總體思路。
3. 對 PHP 寫入的內容強制授予特定的權限。 這可以通過 src / config.php 檔案中帶有如下一行的程式碼來完成： `$config['chmodWritableValue'] = 0666;` 這種方法對於開發目的來說可能是最簡單的。

請注意，如果您正在開發附加元件，則可能會有其他需要由 CLI 和 Web 伺服器用戶寫入的位置。 值得注意的是，這包括附加元件中的 `_output` 目錄。 在這種情況下，以您的 CLI 用戶身份運行 Web 伺服器可能會減少摩擦。 
如果您採用其他任何方法，則可能需要確保您的 Web 伺服器可以寫入整個 XenForo 安裝；在生產中不建議這樣做。                              

## 安裝

當前安裝 XF 2.0 的方式是通過新的 CLI 系統。 許多開發過程只能使用 CLI 來進行，所以我們就來使用它來安裝 XF 2.0。 要運行這些指令，您將需要訪問 終端/shell，使用 php CLI 指令和當前工作目錄應該是您上傳 XF 2.0 檔案的根目錄。

!!! Warning
	為消除檔案權限問題，我們建議以與 PHP 在 web 伺服器上執行的相同使用者來執行安裝程式。 如果不這樣做，則應該採取措施確保權限設定正確。 有關更多詳細信息，請參見上面的章節。

要開始安裝，只需輸入以下指令：

!!! terminal
    *$* php cmd.php xf:install

您會被問到許多問題，例如初始管理員用戶名和密碼，板塊標題。 此後，將匯入 XF 2.0 資料庫表和主要的資料。

XF 2.0 現在已安裝完成！

## 重新安裝

有時可能需要重新安裝 XF2。 在不支持升級的 "開發預覽" 階段尤其如此。 如果準備好重新安裝，請按照上面的 [下載 XF 2.0](#downloading-xf-20) 部分下載新檔案（如果適用）。 一般來說，應該可以合併和覆蓋現有檔案。 如果要進行完全乾淨的重新安裝，則可能需要保存 config.php 檔案的副本，或者按照 [創建 src/config.php](#creating-srcconfigphp) 中的說明重新創建它。

在上傳新檔案之前，您應該刪除 `data` 和 `internal_data` 目錄的內容。

最後，您只需要開始安裝即可，與上面類似。 您將需要使用此 `--clear` 選項來刪除所有現有的 xf_ 資料表。

!!! terminal
    *$* php cmd.php xf:install --clear

重新安裝完成後，您現在應該可以重新登錄了。

如果您正在開發附加元件，並且選擇保留或備份現有 `src/addons` 目錄，則可以使用 [匯入開發輸出](/development-tools/#_15) 指令來還原附加元件資料。

!!! warning
    如果選擇備份和還原 `src/addons` 目錄，請小心。 其中的 `XF` 目錄包含 XF 主要的資料，因此不應從備份中還原，以確保您始終擁有最新版本的檔案。

    以這種方式執行重新安裝是破壞性的操作，它將刪除您創建的所有資料。 此外，請記住，僅帶有 `xf_` 前綴的資料表才會被清除。 這也是建議所有資料表（甚至是附加元件）都應該以 `xf_` 為前綴的一個重要原因。

## 驗證檔案完整性

當您安裝 XF2 時，我們會在安裝過程中執行檔案完整性檢查。 如有必要，並且您無法通過 Admin CP 中的頁面執行檢查，則可以運行 CLI 指令來執行該檢查。

!!! terminal
    *$* php cmd.php xf:file-check _[addon_id]_

如果您希望對所有檔案（包括 XF 本身）進行檔案運行健康檢查，只需省略 `[addon_id]` 參數即可。 僅對於 XF，僅使用 `XF` 代替該參數，或者對於特定的附加元件，只需指定您要檢查的附加元件 ID。

## Add-on 管理指令

除了上述用於安裝 XF2 的指令之外，還有一些用於管理附加元件的指令。

### 安裝

!!! terminal
    *$* php cmd.php xf:addon-install _[addon_id]_

安裝指定的附加元件，只要它是可用的，並通過檔案運行健康檢查。 如果有可用的開發輸出，您將被要求確認是否要將其用於安裝，而不是匯出的資料 XML 檔案。

### 升級

!!! terminal
    *$* php cmd.php xf:addon-upgrade _[addon_id]_

只要指定的附加元件是可升級的，就對其進行升級，並通過檔案運行健康檢查。 可以選擇從開發輸出中進行匯入。

### 重建

!!! terminal
    *$* php cmd.php xf:addon-rebuild _[addon_id]_

只要指定的附加元件是可重建的，就為其重建主要的資料，並通過檔案運行健康檢查。 這將重新匯入附加元件的資料。 可以選擇性地執行從開發輸出中匯入。

### 解除安裝

!!! terminal
    *$* php cmd.php xf:addon-uninstall _[addon_id]_

解除安裝指定的附加元件，只要它是不可解除安裝的。
