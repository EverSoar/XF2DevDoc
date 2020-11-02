# 資料實體、查找器、儲存庫

在 XF2 中，有許多與資料互動的方法。 在 XF1 中，這主要是針對在 Model 檔案中寫出原始 SQL 語句。 XF2 中的方法已經脫離了這一點，我們增加了一些新的方法。 我們首先來看一下執行資料庫查詢的首選方法 - Finder。

## Finder (查找器系統)

我們引入了一個新的 "Finder" 系統，它允許以物件導向的方式以編程方式建立查詢，這樣就不需要編寫原始資料庫查詢。 Finder 系統與 Entity 系統攜手合作，我們在下面詳細介紹。 傳入 finder 方法的第一個參數是你要處理的 Entity 的短類名。 我們就把上面一節中提到的一些查詢轉換為使用 finder 系統來代替。 例如，要訪問單個用戶記錄：

```php
$finder = \XF::finder('XF:User');
$user = $finder->where('user_id', 1)->fetchOne();
```

直接查詢方法與使用 Finder 之間的主要區別之一是 Finder 返回的資料基本單位不是一個陣列。 在 Finder 物件呼叫 `fetchOne` 方法的情況下（該方法只從資料庫中返回單個 row ），將返回單個 Entity 物件。

讓我們看看一個稍微不同的方法，它將返回多個 row：

```php
$finder = \XF::finder('XF:User');
$users = $finder->limit(10)->fetch();
```

這個例子將從 xf_user 資料表中查詢 10 條記錄，並以 `ArrayCollection` 物件的形式返回。 這是一個特殊的物件，它的作用類似於一個陣列，因為它是可以遍歷的（你可以在它裡面循環），而且它有一些特殊的方法，可以告訴你它所擁有的條目總數，通過某些值進行分組，或者其他類似於陣列的操作，如過濾、合併、獲取第一個或最後一個條目等。

Finder 查詢，一般應該是期望從資料表中檢索所有的 Column，因此，沒有特定的對應關係僅獲取某些 Column 中的某些值。

相反，如果要獲得單個值，你只需要獲取一個 Entity，然後直接從這個 Entity 中讀取值：

```php
$finder = \XF::finder('XF:User');
$username = $finder->where('user_id', 1)->fetchOne()->username;
```

同樣地，如果要從 Column 中獲取一個陣列的值，可以使用 `pluckFrom` 方法：

```php
$finder = \XF::finder('XF:User');
$usernames = $finder->limit(10)->pluckFrom('username')->fetch();
```

到目前為止，我們已經看到 Finder 應用了一些簡單的 where 和 limit 約束。 所以我們來看看 Finder 的更多詳細內容，包括關於 `where` 方法本身的更多細節。

### where 方法

`where` 方法最多可以支援三個參數。 第一個參數是條件本身，例如你要搜尋的 Column。 第二個參數通常是運算子。 第三個是被搜尋的 value。 如果你只提供了兩個參數，就像你在上面看到的那樣，那麼它自動意味著運算子是 `=`。 以下是其他有效的運算子列表：

* `=`
* `<>`
* `!=`
* `>`
* `>=`
* `<`
* `<=`
* `LIKE`
* `BETWEEN`

所以，如果我們想要得到最近 7 天內註冊的有效用戶名單：

```php
$finder = \XF::finder('XF:User');
$users = $finder->where('user_state', 'valid')->where('register_date', '>=', time() - 86400 * 7)->fetch();
```

正如你所看到的，你可以隨意呼叫 `where` 方法，但除此之外，你還可以選擇傳入一個陣列做為該方法的唯一參數，並在一次呼叫中建立你的條件。 陣列方法支援兩種類型，我們可以在上面建立的查詢中使用這兩種類型：

```php
$finder = \XF::finder('XF:User');
$users = $finder->where([
    'user_state' => 'valid',
    ['register_date', '>=', time() - 86400 * 7]
])
->fetch();
```

通常不建議或明確地這樣混合使用，但它確實在一定程度上展示了該方法的靈活性。 現在條件已經在一個陣列中，我們可以指定 Column 名（作為陣列的 key）和隱式的 `=` 運算子的 value，或者我們可以實際定義另一個包含 column、運算子和 value 的陣列。

### whereOr 方法

在上面的例子中，兩個條件都需要滿足，即每個條件都由 `AND` 運算子連接。 然而，有時需要只滿足部分條件，這可以通過使用 `whereOr` 方法來實現。 例如，如果你想搜尋那些無效的用戶或者是發佈了零則留言的用戶，你可以建立如下：

```php
$finder = \XF::finder('XF:User');
$users = $finder->whereOr(
    ['user_state', '<>', 'valid'],
    ['message_count', 0]
)->fetch();
```

與上一節的例子類似，除了傳遞最多兩個條件作為單獨的參數外，你也可以只傳遞一個條件陣列到第一個參數：

```php
$finder = \XF::finder('XF:User');
$users = $finder->whereOr([
    ['user_state', '<>', 'valid'],
    ['message_count', 0],
    ['is_banned', 1]
])->fetch();
```

### with 方法

`with` 方法本質上等同於使用 `INNER|LEFT JOIN` 語句，儘管它依賴於 Entity 已經定義了它的 "Relation"。 我們在下一頁才會討論這個問題，但這應該只是讓你了解它是如何運作的。 現在讓我們使用主題 finder 來檢索一個特定的主題：

```php
$finder = \XF::finder('XF:Thread');
$thread = $finder->with('Forum', true)->where('thread_id', 123)->fetchOne();
```

這個查詢將獲取 `thread_id = 123` 的主題實體，但它也會在背景與 xf_forum 資料表進行 join。 在控制如何做 `INNER JOIN` 而不是 `LEFT JOIN` 方面，這就是第二個參數的作用。 在本例中，我們將 "must exist" 參數設定為 true，所以它會把連接語句轉換為使用 `INNER` 而不是預設的 `LEFT`。

我們將在下一節詳細介紹如何訪問從這個 join 獲取的資料。

也可以在 `with` 方法中傳遞一個 Relation 陣列來進行多重 join。

```php
$finder = \XF::finder('XF:Thread');
$thread = $finder->with(['Forum', 'User'], true)->where('thread_id', 123)->fetchOne();
```

這將會 join 到 xf_user 資料表以獲得主題作者。 然而，由於第二個參數仍然是 `true`，我們可能不需要為 User join 做一個 `INNER` join，所以，我們可以用 chain 方法代替：

```php
$finder = \XF::finder('XF:Thread');
$thread = $finder->with('Forum', true)->with('User')->where('thread_id', 123)->fetchOne();
```

### order, limit 和 limitByPage 方法

#### order 方法

這個方法允許你按照特定的順序來修改你的查詢獲取結果。 它需要兩個參數，第一個是 column 名，第二個是可選的排序方向。 所以，如果你想列出擁有最多留言的 10 個用戶，你可以建立像這樣的查詢：

```php
$finder = \XF::finder('XF:User');
$users = $finder->order('message_count', 'DESC')->limit(10);
```

!!! note
    現在可能是一個很好的時機來提及，finder 方法大多可以按照任何順序呼叫。 例如： `$threads = $finder->limit(10)->where('thread_id', '>', 123)->order('post_date')->with('User')->fetch();` 雖然如果你按照這個順序寫一個 MySQL 查詢，肯定會遇到一些語法問題，但 Finder 系統還是會按照正確的順序來構建，上面的程式碼雖然看起來很奇怪，可能也不建議使用，但那樣寫確實是完全正確的。

與標準的 MySQL 查詢一樣，可以對多個 Column 的結果集進行排序。 要做到這一點，你可以再次呼叫 order 方法。 
也可以使用陣列將多個 order 子句傳遞到 order 方法中。

```php
$finder = \XF::finder('XF:User');
$users = $finder->order('message_count', 'DESC')->order('register_date')->limit(10);
```

#### limit 方法

我們已經看到了如何將查詢 limit 在特定數量的記錄上：

```php
$finder = \XF::finder('XF:User');
$users = $finder->limit(10)->fetch();
```

不過，其實還有一個辦法可以直接呼叫 limit 方法：

```php
$finder = \XF::finder('XF:User');
$users = $finder->fetch(10);
```

可以直接將你的 limit 傳入 `fetch()` 方法。 另外值得注意的是，`limit`（和 `fetch`）方法支援兩個參數。 第一個顯然是 limit，第二個是 offset。

```php
$finder = \XF::finder('XF:User');
$users = $finder->limit(10, 100)->fetch();
```

這裡的 offset 基本上意味著前 100 個結果將被棄用，之後的前 10 個結果將被返回。 這種方法對於提供分頁結果是很有用的，不過我們其實也有一個更簡單的方法...

#### limitByPage 方法

這個方法是一種輔助方法，最終會根據你目前檢視的 "頁面" 和你所需要的 "每頁" 數量來設定相應的 limit 和 offset。

```php
$finder = \XF::finder('XF:User');
$users = $finder->limitByPage(3, 20);
```

在這種情況下，limit 將被設定為 20（這是我們每頁的值），offset 將被設定為 40，因為我們從第 3 頁開始。

偶爾，我們需要抓取超過 limit 的額外資料。 Over-fetching 可以幫助偵測你在目前頁面之後是否有額外的資料要顯示，或者你是否需要根據權限過濾設定下來的初始結果。 我們可以通過第三個參數來實現：

```php
$finder = \XF::finder('XF:User');
$users = $finder->limitByPage(3, 20, 1);
```

這樣一來，從第 3 頁開始，總共最多可獲得 **21個** 用戶（20+1）。

### getQuery 方法

當你第一次開始使用 finder 時，儘管它很直觀，但你可能偶爾會想知道你是否正確地使用了它，以及它是否會建立你期望的查詢。 我們有一個名為 `getQuery` 的方法，它可以告訴我們目前的 finder 物件將建立的查詢。 例如：

```php
$finder = \XF::finder('XF:User')
	->where('user_id', 1);

\XF::dumpSimple($finder->getQuery());
```

這將輸出類似於以下的語句：

```plain
string(67) "SELECT `xf_user`.*
FROM `xf_user`
WHERE (`xf_user`.`user_id` = 1)"
```

你可能不會經常需要它，但如果 finder 沒有返回你所期望的結果，那它可能會很有幫助。 在 [Dump 變數](/development-tools/#dump) 部分閱讀更多關於 `dumpSimple` 方法的內容。

### 自定義 finder 方法

到目前為止，我們已經看到 finder 物件被設定了一個類似於 `XF:User` 和 `XF:Thread` 的參數。 在大多數情況下，這標識了 finder 正在處理的 Entity 類，並將解析為，例如，`XF/Entity/User`。 然而，它也可以另外代表一個 finder 類。 finder 類是可選的，但它們可以作為一種方法來為特定的 finder 類型新增自定義 finder 方法。 為了看到這一點，讓我們看看與 `XF:User` 相關的 finder 類，它可以在 `XF\FinderUser` 類中找到。

下面示例是該 class 中的一個 finder 方法：

```php
public function isRecentlyActive($days = 180)
{
	$this->where('last_activity', '>', time() - ($days * 86400));
	return $this;
}
```

現在，我們可以在任何 User finder 物件上呼叫該方法。 所以，如果我們拿前面的一個例子來說：

```php
$finder = \XF::finder('XF:User');
$users = $finder->isRecentlyActive(20)->order('message_count', 'DESC')->limit(10);
```

這個查詢，之前只是按照留言數降序返回 10 個用戶，現在將按這個順序返回最近 20 天內活躍的 10 個用戶。

儘管對於很多 Entity 類型來說，finder 類是不存在的，但仍然可以通過 [繼承類](/general-concepts/#_5) 一節中提到的相同方式來繼承這些不存在的類。

## The Entity system (實體系統)

如果您熟悉 XF1，您可能會熟悉 Entity 背後的一些概念，因為它們最終是從那裡的 DataWriter 系統衍生出來的。 如果您對它們不是很熟悉，下面的部分應該會讓您有所了解。

### Entity structure (實體結構)

`Structure` 物件由一些屬性組成，這些屬性定義了 Entity 的 Structure 和它所涉及的資料表。 Structure 物件本身就設定在它所涉及的 Entity 內部。 我們來看看 User Entity 中的一些常用屬性：

#### Table (資料表)
```php
$structure->table = 'xf_user';
```
它說明了 Entity 在 Update 和 Insert 記錄時使用哪張資料表，也告訴我們 Finder 在構建查詢執行時從哪張資料表讀取。 此外，它還知道你的查詢需要 join 到哪些其他資料表中起到了作用。

#### Short name (簡短名)
```php
$structure->shortName = 'XF:User';
```
這只是 Entity 本身和 Finder 類（如果適用）的短類名。

#### Content type (內容類型)
```php
$structure->contentType = 'user';
```
這定義了這個 Entity 代表的內容類型。 在大多數 Entity 結構中不會需要這個。 它用於連接到 "content type" 系統使用的特定事物（這將在另一節中介紹）。

#### Primary key (主鍵)
```php
$structure->primaryKey = 'user_id';
```
定義了代表資料表中 Primary key 的 Column，如果一個資料表支援多個 Column 作為 Primary key，那麼可以定義為陣列。 

#### Columns (多行資料)
```php
$structure->columns = [
    'user_id' => ['type' => self::UINT, 'autoIncrement' => true, 'nullable' => true, 'changeLog' => false],
    'username' => ['type' => self::STR, 'maxLength' => 50,
        'required' => 'please_enter_valid_name'
    ]
    // 還有更多的 columns...
];
```
這是 Entity 設定的關鍵部分，因為這裡面會詳細解釋 Entity 負責的每個資料庫 column 的具體情況。 這告訴我們預期的資料類型，是否需要一個值，它應該匹配什麼格式，是否應該是唯一值，它的預設值是什麼，等等。

根據 `type`，Entity 管理器知道是否以某種方式對一個值進行編碼或解碼。 這可以是一個簡單的過程，將一個值轉換為一個字串或整數，或者稍微複雜一點，例如在向資料庫寫入時，對一個陣列使用 `json_encode()`，或者在從資料庫讀取時，對一個 JSON 字串使用 `json_decode()`，以便將該值作為一個陣列正確地返回給 Entity 物件，而不需要我們手動這樣做。 它還可以支援對逗號分隔的值進行適當的編碼/解碼。

偶爾需要在寫入一個值之前對其進行一些額外的驗證或修改。 舉個例子，在 User Entity 中，請看 `verifyStyleId()` 方法。 當在 `style_id` 欄位上設定一個值時，我們會自動檢查是否存在一個名為 `verifyStyleId()` 的方法，如果存在，我們會先通過該方法執行該值。

#### Behaviors (行為)
```php
$structure->behaviors = [
    'XF:ChangeLoggable' => []
];
```
這是該 Entity 應使用的 Behaviors 類陣列。 Behaviors 類是一種允許某些程式碼在多種 Entity 類型中通用重複使用的方式（僅在 Entity 變化時，而不是在讀取時）。 一個很好的例子是 `XF:Likeable` Behaviors，它能夠對支援可被 "點讚" 的內容實體自動執行某些操作。 這包括當內容中發生可見性變化時自動重新計數，以及當內容被刪除時自動刪除點讚數。

#### Getters (獲取器)
```php
$structure->getters = [
    'is_super_admin' => true,
    'last_activity' => true
];
```
當呼叫 name 欄位時，會自動呼叫 Getter 方法。 例如，如果我們從 User Entity 中請求 `is_super_admin`，就會自動檢查並使用 `getIsSuperAdmin()` 方法。 有趣的是，xf_user 資料表實際上並沒有一個名為 `is_super_admin` 的欄位。 這個欄位實際上存在於 Admin Entity 中，但我們將其作為一個 getter 方法新增到了資料表中，作為訪問該值的一種簡寫方式。 Getter 方法也可以用來直接覆蓋現有欄位的值，這裡的 `last_activity` 值就是如此。 `last_activity` 實際上是一個快取值，通常在用戶登出時才會更新。 然而，我們在 xf_session_activity 資料表中儲存了用戶最新的活動日期，所以我們可以使用 `getLastActivity` 方法來返回該值而不是快取的最後活動值。 如果你需要完全繞過 getter 方法，只需要得到真正的 Entity 值，只需要在 column 名後加上下劃線，例如 `$user->last_activity_`。

因為一個 Entity 就像其他的 PHP 物件一樣，你可以給它們新增更多的方法。 一個常見的使用案例是新增一些像權限檢查這樣的方法，這些方法可以在 Entity 自身上面呼叫。

#### Relations (關聯)
```php
$structure->relations = [
    'Admin' => [
        'entity' => 'XF:Admin',
        'type' => self::TO_ONE,
        'conditions' => 'user_id',
        'primary' => true
    ]
];
```
這就是 Relations 的定義。 什麼是 Relations？ 它們定義了 Entity 之間的關聯，這些關聯可以被用來對其他資料表執行 join 查詢，或者快速獲取與 Entity 相關的記錄。 如果我們還記得 finder 中的 `with` 方法，假如我們想獲取一個特定的用戶，並預先獲取該用戶的 Admin 記錄（如果它存在的話），那麼我們將做如下操作：

```php
$finder = \XF::finder('XF:User');
$user = $finder->where('user_id', 1)->with('Admin')->fetchOne();
```
 
這將使用 User Entity 中定義的 `Admin` 相關的資訊和 `XF:Admin` Entity Structure 的細節，知道這個用戶查詢應該在 xf_admin 資料表和 `user_id` column 上執行 `LEFT JOIN`。 要從 User Entity 中獲取 Admin 最後登錄日期，請執行以下操作：

```php
$lastLogin = $user->Admin->last_login; // 返回最後一次 Admin 登錄的時間戳
```

然而，並不總是需要在 finder 中進行 join 來獲取 Entity 的相關資訊。 例如，如果我們以上面的例子為例，不呼叫 `with` 方法：

```php
$finder = \XF::finder('XF:User');
$user = $finder->where('user_id', 1)->fetchOne();
$lastLogin = $user->Admin->last_login; // 返回最後一次 Admin 登錄的時間戳
```

我們在這裡仍然得到 `last_login` 值。 它是通過執行額外的查詢來快速獲取 Admin Entity 的。

上面的例子使用了 `TO_ONE` 類型，因此，這種關聯將一個 Entity 與另一個 Entity 聯繫起來。 我們還有一個 `TO_MANY` 類型。

它不可能獲取整個 `TO_MANY` 關聯（例如在 finder 上使用 join / `with` 方法），但以查詢為代價，它可以在任何時候快速讀取，例如在上面最後的 `last_login` 例子中。

在 User Entity 上定義的一個這樣的 Relation 是 `ConnectedAccounts` 關聯：

```php
$structure->relations = [
    'ConnectedAccounts' => [
    	'entity' => 'XF:UserConnectedAccount',
    	'type' => self::TO_MANY,
    	'conditions' => 'user_id',
    	'key' => 'provider'
    ]
];
```

這個 relation 能夠從 xf_user_connected_account 資料表中返回與目前用戶 ID 相匹配的記錄，作為一個 `FinderCollection`。 這類似於我們在上面 [Finder](#finder) 部分提到的 `ArrayCollection` 物件。 在 Relation 定義中，指定該集合應該由 `provider` 欄位作為 key。

雖然在執行 finder 查詢時不可能獲取多條記錄，但可以使用 `TO_MANY` 關聯從該 Relation 中獲取 **單個** 記錄。 舉個例子，如果我們想查看用戶是否與特定的關聯帳戶提供商相關聯，我們至少可以在查詢時獲取該記錄：

```php
$finder = \XF::finder('XF:User');
$user = $finder->where('user_id', 1)->with('ConnectedAccounts|facebook')->fetchOne();
```

#### Options (選項)
```php
$structure->options = [
	'custom_title_disallowed' => preg_split('/\r?\n/', $options->disallowedCustomTitles),
	'admin_edit' => false,
	'skip_email_confirm' => false
];
```
Entity Option 是在某些條件下修改 Entity Behavior 的一種方式。 例如，如果我們將 `admin_edit` 設定為 true（在 Admin CP 中編輯用戶時就是如此），那麼某些檢查將被跳過，例如允許用戶的電子郵件地址為空。

### Entity 生命週期

Entity 在管理資料庫內記錄的生命週期方面扮演著重要角色。 除了從它那裡讀取值，向它寫入值之外，Entity 還可以用來刪除記錄，並在所有這些操作發生時觸發某些事件，從而可以執行某些任務，或者也可以更新某些相關記錄。 讓我們來看看 Entity 在儲存時發生的一些事件：

* `_preSave()` - 在儲存過程開始之前發生，主要用於執行任何其他儲存前的驗證或在儲存發生之前設定其他資料。
* `_postSave()` - 在資料被儲存後，但在任何交易被提交之前，將呼叫此方法，您可以使用此方法來執行 Entity 被儲存後應觸發的任何其他事。
                  
另外還有 `_preDelete()` 和 `_postDelete()`，它們的運作方式類似，但在刪除發生時。

Entity 還能夠提供有關它目前狀態的資訊。 例如，有一個 `isInsert()` 和 `isUpdate()` 方法，這樣你就可以檢測到這是一個新記錄被插入還是一個現有記錄被更新。 還有一個 `isChanged()` 方法，可以告訴你自上次儲存後，某個特定欄位是否發生了變化。

讓我們看看這些方法在 User Entity 中的一些實際例子。
 
```php
 protected function _preSave()
 {
 	if ($this->isChanged('user_group_id') || $this->isChanged('secondary_group_ids'))
 	{
 		$groupRepo = $this->getUserGroupRepo();
 		$this->display_style_group_id = $groupRepo->getDisplayGroupIdForUser($this);
 	}
 	
 	// ...
 }
 
 protected function _postSave()
 {
    // ...
    
 	if ($this->isUpdate() && $this->isChanged('username') && $this->getExistingValue('username') != null)
 	{
 		$this->app()->jobManager()->enqueue('XF:UserRenameCleanUp', [
 			'originalUserId' => $this->user_id,
 			'originalUserName' => $this->getExistingValue('username'),
 			'newUserName' => $this->username
 		]);
 	}
 	
 	// ...
```
 
在 `_preSave()` 的例子中，我們根據用戶改變後的用戶組來獲取並快取新的顯示 Group ID。 在 `_postSave()` 的例子中，我們在用戶的名字被更改後觸發一個作業來執行。

## Repository (儲存庫)

對於 XF2 來說，Repository 是一個新的概念，但是如果把它們與 XF1 中的 "Model" 物件進行比較，你可能不會受到指責。 我們在 XF2 中沒有 Model 物件，因為我們有更好的地方和方法來獲取和寫入資料到資料庫。 所以，與其說我們有一個龐大的 class，其中包含了你的附加元件所需要的所有查詢，以及各種不同的方法來操作這些查詢，不如說我們有 finder，它增加了更多的彈性。

另外值得注意的是，在 XF1 中，Model 物件對於許多東西來說有點像 "dumping ground"。 其中很多東西現在都是多餘的。 例如，在 XF1 中，所有的權限重建程式碼都在權限 Model 中。 在 XF2 中，我們有特定的服務和物件來處理這個問題。

那麼，什麼是 Repository 呢？ 它們對應著一個 Entity 和一個 Finder，並持有方法，一般會返回一個為特定目的設定的 Finder 物件。 為什麼不直接返回 Finder 查詢的結果呢？ 好吧，如果我們返回 Finder 物件本身，那麼它可以作為一個有用的繼承點，讓附加元件在 Entity 或集合返回之前，對其進行繼承並修改 Finder 物件。

Repository 也可以包含一些具體的方法，比如快取重建。
