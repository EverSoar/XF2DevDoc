# 路由基礎知識

在 PHP 應用程式中，像是 XF2，我們需要一種能夠接受用戶對特定 URL 的請求的方法，了解該 URL 代表的 controller、 action 和資料，以便向用戶提供適當的響應。 將一個 URL 轉換為程式碼中的一個位置的概念被稱為 "路由"。

在 XF2 中，路由選擇幾乎完全由 Admin CP 中的一個位置管理。 該位置是 `Admin CP > Development > Routes`。 路由按兩種類型之一分組，即 Public 和 Admin 類型，它們分別在 Public 和 Admin 應用程式中提供請求的路由。

## 簡單的例子

在路由頁面 (見上文)，你應該看到一個 `account/` 的條目。 這是一個 Public 路由，提供了對 URL `index.php?account/` 的請求的路由。 這個路由非常簡單，它只包含了少量的配置。 值得注意的是，它由一個 "路由前綴"，一個 Section context 和一個 Controller class 組成。 讓我們更詳細地了解這些部分：

### 路由前綴

路由前綴主要是指在 `index.php?` 之後和第一個 `/` 之前的位置。 它是確定路由請求給哪個 Controller 的第一步。
 
### Section context

此 Section context 告訴 XF 中的導覽系統，當訪問者瀏覽由該路由轉到的頁面時，應該選擇哪個導覽項。 對於 Public 路由，Section context 應該是最高級別導覽項的 ID。 對於 Admin 路由，應該是最特定的管理導覽條目的 ID（無論深度）。

如果是帳號路由，預設情況下，Section context 不一定適用，因為我們沒有 "account" 導覽標籤。 但是，要想看到這個 action，只需將這裡的 "Section context" 值改為 "forums"，儲存更改並在前端進入你的帳號。 現在，你應該看到 "Forums" 導覽標籤被選中了！

### Controller

這是當請求與這個路由匹配時，應該被呼叫的 Controller 的 class 名。 在 "account/" 路由中，我們指定了 `XF:Account`。 這將載入 Account 控制器。 （更多資訊請參見 [短類名](/general-concepts/#_4)）。 程式碼位於以下位置 `src/XF/Pub/Controller/Account.php`。 注意短類名是如何解析為 "infix" 以及前綴 (XF) 和後綴 (Account) 的。 在本例中，這個控制器的下標 (Pub) 是由 Account 路由類型 (public) 推斷出來的。

## Controller action

上面我們解釋了如何將路由匹配到一個特定的 Controller，但我們還不知道如何呼叫該 Controller 內的特定 action。 Controller 本質上是一個包含了許多 action 方法的 class，URL 中 [路由前綴](#_3) 後面的部分表示 Controller 的 action。 給定一個 URL 為 `index.php?account/account-details`，你應該被路由到 `XF/Pub/Controller/Account` class 和 `actionAccountDetails()` 方法。 如果路由沒有指定 action，那麼呼叫的方法只是 `actionIndex()`。

你可以在 [控制器基礎知識](/controller-basics) 部分閱讀更多關於控制器的內容。

## 更進階的例子（路由格式）

讓我們看看 `members/` 路由。 這個路由和 `account/` 路由一樣，還是很簡單的，但是它多了一個欄位："路由格式"。 要了解它是如何工作的，看看你自己在前端的用戶個人資料。 該配置檔案的 URL 看起來會是這樣的 `index.php?members/your-name.1`。具體而言，請注意 `your-name.1` 的部分。 這是我們試圖使用 "路由格式" 來匹配的部分。

"路由格式" 允許我們從請求 URL 中提取資料，因此我們可以將該資訊傳遞到 Controller action 中，以便該 action 可以載入特定的資訊；在這種情況下，它載入請求的用戶個人資料的詳細資訊。 它還可以幫助我們從傳遞進來的資料中建立連結。 下面是語法：
 
```plain
:int<user_id,username>/:page
```
 
有趣的是，此時要注意的是，個人資料 URL 中用於查詢個人資料的重要部分其實並不是 `your-name` 這一點，而實際上是用戶ID（`1`）。 為了證明這一點，改變 URL，用 `not-your-name` 代替 `your-name`。 你會看到找到了正確的個人資料，並且重新定向到了正確的 URL。

上面的格式表明它是一個基於整數的參數。 在建立外部鏈結時，我們從傳入資料的 user_id key 中提取整數。 如果資料中傳遞了一個用戶名，它將被 "slugified"，就像你在你的個人資料的 URL 中看到的整數 ID 一樣。 對於匹配傳入的 URL，這將變成一個匹配整數參數格式的正規化表達式。

`:page` 是用於生成連結的 page-123 部分的快捷方式。 在這種情況下，它在連結參數中尋找頁面。 如果找到了，它就會被放在 URL 中，然後從參數中刪除。 對於傳入的解析，如果匹配（它可以是空的），它將把頁碼新增到傳遞給 Controller 的參數中。

## 路由參數
 
當路由與特定的 Controller 和 action 匹配時，URL 中的任何參數都會被封裝到一個特殊的物件中，我們稱之為 `ParameterBag`。 這個物件專門用於將普通 URL 參數與來自路由匹配的 URL 參數分開的。 `ParameterBag` 物件被傳遞到每個 Controller action 中，使用方法如下：

```php
$userId = $params->user_id;
```
 
## Sub-names
 
也可以將路由分割成更多的 Sub-names。 你可以通過 `members/following` 路由來看到這個 action。 在這個例子中，`following` 是路由 `members` 的 Sub-names。 通常情況下，看起來像 `index.php?members/following` 的 URL，`following` 的部分將表示 action，並簡單地與普通的 `members/` 路由進行匹配。 但是，如果有一個路由匹配了前綴 "members" 和後面的 "Sub-names"，就會被使用。 這裡也是如此，所以它建立了一個像是下面的連結：

```plain
members/:int<user_id,username>/following/:page
```

對於傳入路由的匹配，該路由將在基本成員路由之前進行測試；如果匹配，則使用該路由。

這種 sub-name 系統允許行為改變，例如改變參數的位置或者將路由分組到不同的 Controller 中或使用不同的參數。 您可以在資源管理器和媒體庫附加元件中看到後者的例子。