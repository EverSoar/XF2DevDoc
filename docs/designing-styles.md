# 設計樣式

在 XF2 中，我們引入了一種全新的方式來建立和編輯樣式，稱為 "設計師模式"。 設計師模式是一個 CLI 工具集合，允許您直接在檔案系統中修改樣式中的某些模板。 它還輸出各種 metadata 和樣式屬性資訊，這對版本控制和協同合作非常有幫助。

## 開啟設計師模式

開啟設計師模式的第一步是在 `config.php` 中啟用它。

```php
$config['designer']['enabled'] = true;
```

您也可以選擇為設計器模式檔案指定不同的路徑存在於檔案系統中。 以下是預設路徑。 要改變路徑，請在你的 `config.php` 中新增以下內容，並相應修改路徑。

```php
$config['designer']['basePath'] = 'src/styles';
```

## 開啟樣式設計師模式

設計師模式必須為每個樣式明確啟用。 我們通過使用 CLI 和指定樣式的樣式 ID，並選擇 "設計師模式 ID" 來啟用樣式的設計師模式：

!!! terminal
    *$* php cmd.php xf-designer:enable _[style_id]_ _[designer_mode_id]_
    
設計師模式 ID 是未來與設計師模式相關的指令中使用的識別符號。 一旦開啟，目前修改後的樣式元件將被匯出到 `[basePath]/[designer_mode_id]` 目錄中。

當開啟該樣式的設計師模式時，如果該目錄已經存在，你會有一個選擇，即我們是否應該從樣式中覆蓋該目錄的當前內容，或者是否應該從該目錄的當前內容中覆蓋目前樣式。
 
## 關閉樣式設計師模式

要關閉某個樣式的設計師模式，您只需執行以下 CLI 指令。

!!! terminal
    *$* php cmd.php xf-designer:disable _[designer_mode_id]_
    
預設情況下，這將在檔案系統中保留設計師模式的輸出副本。 要刪除資料，你可以用 `--clear` 選項執行同樣的指令。

!!! terminal
    *$* php cmd.php xf-designer:disable _[designer_mode_id]_ --clear
    
## 什麼是輸出，在哪裡輸出？

重要的是要記住，XF 中的樣式只由在該樣式中 **修改的內容** 組成。 這意味著設計師模式的輸出將只包括在樣式中被修改的內容。 在父樣式中被修改的模板和樣式屬性不會被輸出。

### 模板

模板將被輸出到 `[basePath]/[designer_mode_id]/templates` 目錄中。 在該目錄中，您可以為每種類型（如 admin、email 和 public）設定另一個目錄。

模板將以 HTML 格式輸出，並可在檔案系統中直接編輯。 在檔案系統上所做的更改會在該模板載入到頁面時被匯入並編譯。 同樣，您也可以通過從檔案系統中刪除模板（如果它以前被修改過）來恢復它。

### 樣式屬性和群組

樣式屬性和群組將被輸出到 `[basePath]/[designer_mode_id]/style_properties` 和 `[basePath]/[designer_mode_id]/style_property_groups` 目錄下。 它們以 JSON 格式匯出，作為一種有用的方式，通過版本控制系統監控這些檔案的變化。
 
不建議直接修改這些檔案，因為對它們的修改 **不會** 像模板那樣自動匯入。

## 修改特定模板

考慮到一個樣式只代表在該樣式中被修改的元件，當設計師模式開啟時，檔案系統也將只包含在該樣式中被修改的元件。 這樣就無法輸出每個模板和樣式屬性的有效版本。
 
要將一個模板標記為樣式中已修改的，您可以通過在 Admin CP 中編輯它的常規方式進行。 如果啟用了設計器模式，在 Admin CP 中修改的模板和樣式屬性將自動寫入檔案系統。 然而，使用 CLI 指令來修改或 "touch" 一個模板可能會更方便。

!!! terminal
    *$* php cmd.php xf-designer:touch-template _[designer_mode_id]_ _[template_type:template_title]_
    
只要指定的模板存在於父樣式或主樣式中，它就會被複製到目前樣式中，並輸出到檔案系統中。 然後你可以直接在檔案系統中修改模板。

如果你想在你的樣式中建立一個完全自定義的模板（在樹狀結構中的任何其他樣式中都不存在），你可以使用同樣的指令，但你只需要傳遞 `--custom` 選項：

!!! terminal
    *$* php cmd.php xf-designer:touch-template _[designer_mode_id]_ _[template_type:template_title]_ --custom
    
## 其他有用的指令

還有其他一些與設計師模式有關的有用指令：

### 從資料庫中匯出

當設計師模式被啟用時，這個指令通常會自動執行，但如果出於某種原因，你想用目前資料庫中的內容覆蓋檔案系統副本，那麼你可以執行以下指令：

!!! terminal
    *$* php cmd.php xf-designer:export _[designer_mode_id]_
    
也可以只匯出特定的類型，例如 `xf-designer:export-templates`。

### 從檔案系統匯入

這條指令將用檔案系統中的內容覆蓋資料庫中的樣式副本：

!!! terminal
    *$* php cmd.php xf-designer:import _[designer_mode_id]_
    
也可以只匯入特定的類型，例如 `xf-designer:import-templates`。

### 同步模板

該指令類似於匯入模板（見上文），但它不會覆蓋所有內容，而只會匯入模板，並在 metadata 發生變化時重新編譯。 它還會相應地應用版本號更新。

!!! terminal
    *$* php cmd.php xf-designer:sync-templates _[designer_mode_id]_
    
### 還原模板

這個指令可以用來恢復模板，有效地從目前樣式中刪除自定義版本。

!!! terminal
    *$* php cmd.php xf-designer:revert-template _[designer_mode_id]_ _[template_type:template_title]_
    
也可以通過從檔案系統中刪除模板來觸發還原。