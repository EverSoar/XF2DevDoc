# 控制器基礎知識

在基本層面上，Controller 是當你在 XF 中訪問一個頁面時執行的程式碼。 在基本層面上，Controller 一般負責處理用戶的輸入，並將用戶的輸入傳遞到適當的地方，通常是執行某種資料庫操作（Model）或載入視覺化內容（View）。

當用戶點選連結時，請求的 URL 會被路由導到特定的 Controller 和 Controller action。 參見 [路由基礎知識](/routing-basics)。 例如，在 XF 中，如果你點選一個類似於 `index.php?conversations/add` 的 URL，你將被路由導到 `XF\Pub\Controller\Conversation` Controller 和 `add` action。
 
如果你在檔案系統中查看這個 class（參見 [自動載入器](/general-concepts/#_2)，以了解 class 和檔案路徑如何相互映射的描述），你會注意到有許多方法以 `action` 為前綴命名。 所有這些方法都表示一個特定的 Controller action。 所以，要想在查看上面提到的 conversations/add 頁面時看到相關程式碼，請在這個檔案中查詢 `public function actionAdd()`。

XF Controller 負責返回一個 reply 物件，該物件一般包括以下類型之一：

## View 回應
 
這是 XF 開發過程中最常見的回應之一。 一個返回 View 回應的 Controller 通常需要傳遞多達三個參數。 一個 View class (下面會有更多的介紹)，一個模板名稱，以及一個由 `$viewParams` 組成的陣列，這個陣列是模板應該有的資料。

這裡是一個典型的 Controller action 的例子，它返回一個 View 回應：

```php
public function actionExample()
{
    $hello = 'Hello';
    $world = 'world!';
    
    $viewParams = [
        'hello' => $hello,
        'world' => $world
    ];
    return $this->view('Demo:Example', 'demo_example', $viewParams);
}
```

第一個參數是一個特定 View class 的簡短 class 名。 這個 class 可能存在，也可能不存在（通常情況下，它不需要存在，我們在後面會介紹更多地 View class ），但它應該有一個 Controller 和 action 的大致唯一名稱。 與其他 [短類名](/general-concepts/#_4) 一樣，上面的特定短類名將解析為 `Demo/Pub/View/Example`。 同樣，`Pub` 是由 Controller 類型自動推斷出來的。

第二個參數是模板名稱。 在本例中，我們正在尋找一個名為 `demo_example` 的模板。

第三個參數是 應該提供給 view 模板 參數/值 的陣列。 這個陣列一般應該是一對 `key => value`。 上面的例子是將兩個模板參數傳遞給模板。 陣列中的 `key` 部分表示模板中可用的變數名稱。 陣列中的 `value` 部分表示值。

因此，如果我們在 `demo_example` 模板中有以下內容：

```html
{$hello} {$world}
```

該模板將輸出以下內容：

```plain
Hello world!
```

## 重新導向回應

當你希望在用戶完成某種操作後將其重新導向到不同的 URL 時，會返回這個回應。

一個常見的使用案例是在用戶通過表單提交資料後，你可能希望將他們重新導向到不同的頁面，例如將用戶返回到一個項目清單。

下面是一個典型的 Controller action 的例子，它執行了一個重新導向：

```php
public function actionRedirect()
{
    return $this->redirect($this->buildLink('demo/example'), '這是一個重定向資訊。', 'permanent');
}
```

第一個參數是要重新導向的 URL。 這個例子將把用戶重新導向到 `index.php?demo/example` 的 URL。

第二個參數只有在表單通過 AJAX 請求提交時才會顯示，AJAX 請求選擇防止重新導向。 結果將是一個 "即時訊息"，從螢幕頂部出現您選擇的訊息。 您不必提供您自己的資訊。 如果沒有提供，它將預設為 "您的更改已被儲存"。

第三個參數預設為 `temporary`，但你也可以選擇將其設定為 permanent，就像本例一樣。 這裡唯一的區別是伺服器提供的 HTTP 回應碼的類型。 在大多數情況下，臨時是理想的，這將以 303 碼回應。 `permanent` 將發出 301 回應碼。

雖然你可以通過這種方式觸發一個 permanent 重新導向，但實際上有一個特定的方法，可以使用如下。 它也需要一個 'message' 參數，但和上面一樣，它是可選的。

```php
public function actionRedirect()
{
    return $this->redirectPermanently($this->buildLink('demo/example'));
}
```

## 錯誤回應

顧名思義，這個回應就是你需要向用戶顯示錯誤時返回的內容。 這稍稍簡單，下面是一個例子：

```php
public function actionError()
{
    return $this->error('很不幸，你要找的東西無法找到。', 404);
}
```

這裡只支援兩個參數。 第一個是你想顯示的錯誤資訊，第二個是你想讓伺服器發送的 HTTP 回應碼。 404 將代表在沒有找到東西時的適當回應。

## 訊息回應

這個回應與錯誤回應非常相似，支援相同的參數。 主要的區別是，在外觀上，顯示的資訊不是以錯誤的形式出現。

## 異常回應

有時有必要中斷 Controller 程式碼的正常流程，並用異常回應來代替。 異常回應不一定代表一個錯誤；例如，它們可以用來強制 Controller 執行一個重新導向。 然而，通常情況下，它們通常會被用來中斷 Controller 的流程以顯示一個錯誤，就像下面的例子一樣：

```php
public function actionException()
{
    throw $this->exception($this->error('發生意外錯誤'));
}
```

異常回應只接受一個參數，實際上這個參數必須是其他形式的 Reply 物件，比如 [錯誤回應](#_3)。 這個特殊的例子會拋出一個異常，此時整個控制器的程式碼會停止，並顯示一個標準的錯誤。

請注意，異常回應必須用 `throw` 來 "拋出"，而不是用 `return` 來 "返回"。

## 重定路由回應

在某些情況下，需要將用戶重新導向到一個完全不同的 Controller 或同一 Controller 內的 action，而不需要執行完全的重新導向，不需要改變用戶登陸的 URL，也不需要重複目標 action 的程式碼。

這看起來有點像這樣：

```php
public function actionReroute()
{
    return $this->rerouteController(__CLASS__, 'error');
}

public function actionError()
{
    return $this->error('糟了！出了點問題！');
}
```

在這個特殊的例子中，如果用戶導覽到 `index.php?demo/reroute` URL，他們會看到 `actionError()` 方法的錯誤回應。 他們不會被重新導向，瀏覽器中的 URL 也不會改變，他們只是收到錯誤 action 的回應。

重定路由回應還支持第三個參數，它允許將各種參數從一個 Controller action 傳遞到另一個 Controller。 這可以是一個陣列或一個 `ParameterBag` 物件（稍後將詳細介紹）。

## 修改 Controller Action 回應 （properly）

在 [繼承 class](/general-concepts/#_5) 章節中，我們已經看到了繼承一個 class 是多麼的簡單，但是當繼承一個已經存在的 Controller action 時，需要格外小心。

除非你有特殊的需求，需要完全覆蓋一個現有的 action，並用新的東西來代替它（一般不建議這樣做），否則你應該修改父類的現有回應。 這非常簡單，舉個例子，讓我們修改上面 [view 回應](#view) 例子中的 view 回應。

```php
public function actionExample()
{
    $reply = parent::actionExample();
    
    return $reply;
}
```

假設上面的內容被添加到一個已經存在 `actionExample()` 方法的繼承 Controller 中，上面的內容除了返回原來的 view 回應外，實際上並沒有做任何事情。 現在讓我們把現有的 `hello` 參數改為 "Bonjour" 而不是 "Hello"。

```php
public function actionExample()
{
    $reply = parent::actionExample();
    
    if ($reply instanceof \XF\Mvc\Reply\View)
    {
        $reply->setParam('hello', 'Bonjour');
    }
    
    return $reply;
}
```

因為一個 Controller 回應實際上可以代表許多不同的物件，這些物件具有不同的行為和方法，所以我們必須只嘗試繼承正確的回應類型。 在上面的例子中，我們通過檢查父 `$reply` 物件是否真的是 `View` 類型來做到這一點。 如果我們沒有這樣做，我們繼承了這個 action，而 Controller action 卻用重新導向來回應，那麼很可能會出現錯誤。

在繼承這個動作之前，訪問這個頁面會顯示 "Hello world！"。 繼承後，現在 view 將顯示 "Bonjour world！"。