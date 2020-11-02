# 模板語法

XenForo 2 模板語法對於開發人員和論壇管理員都是強大的工具，使您可以完全控制 XenForo 頁面的排版。

## 最佳示例
- 按照慣例，XenForo 的標籤是 `lowercase`。
- 所有 XenForo 標籤都以 `xf:` 命名空間為前綴。

## 有幫助的訊息

### 對模板進行註釋
如果您想註釋掉一些您不想在最終頁面原始碼中看到的模板代碼（或者勵志的訊息），則可以使用 `xf:comment` 標籤。

```html
<xf:comment>
如果你不再以你喜歡什麼和不喜歡什麼來看待這個世界，
而是看到了事物本身的真實面目，
你就會發現你的生活將會更加地祥和。
</xf:comment>
```

### 在模板中加入另一個模板

`xf:include` 標籤允許你在當前模板中加入一個不同的模板。

```html
<xf:include template="my_template" />
```

只需將 `template` 屬性設定為你想加入的模板名稱即可。


## 模板巨集
模板巨集是 XenForo 模板語法的一個非常強大的部分。

通常，你應該在程式設計語言中使用函數或子程式的任何地方使用巨集。 對於非程式設計師來說，我把這一點總結為：**要麼在**任何您想在多個不同檔案中多次產生相同事物的地方使用巨集，**或者**在不同情況下產生不同的東西（如果你查看定義巨集的指南，這可能會更有幫助）。

!!! warning
	出於可讀性原因，您不應該使用巨集標籤作為變數。 您應該使用 Set 標籤，並像對待任何模板變量一樣對待變量。

### 定義巨集
```html
<xf:macro
    name="my_macro_name">

    <!-- 你的巨集內容 -->

</xf:macro>
```
最簡單的，可以用 `name` 屬性定義一個巨集，並在巨集標籤中重複你想要的內容。

!!! note
	當您在多個檔案中使用巨集時，最佳做法是將巨集放入自己的模板中。

#### 巨集參數
```html
<xf:macro
    name="my_macro_name"
    arg-message="我超厲害的巨集訊息！">

    <h1>訊息</h1>
    <p>{$message}</p>

</xf:macro>
```
在此示例中，為巨集定義了 `arg-message` (`我驚人的巨集訊息！`) 的預設值。 如果使用 message 參數呼叫巨集，則該值將被覆蓋。

有時需要將一個參數標記為必填項。 這可以通過在巨集定義中把參數值設定為 `!` 來實現。

### 加入和使用巨集
```html
<xf:macro template="my_macro_template" name="my_macro_name" />
```
最簡單的方法是，通過設置 `name` 屬性並將標籤保留為空來加入巨集。

!!! note
	使用巨集標籤時，你應該使用標籤的自動關閉形式，以便讓別人更容易區分巨集的定義和用法的區別。

#### 巨集參數
您還可以為巨集提供參數：

```html
<xf:macro template="my_macro_template" name="my_macro_name" arg-argName="argValue" />
```

其中 `argName` 是巨集參數的名稱。

!!! note
	您應該使用 `lowerCamelCase` 來命名你的巨集參數。

## 模板控制結構

XenForo 2 模板語法支援某些控制元件結構，以使某些任務更易於實現。

### If 標籤

if 模板標籤可用於有條件地顯示一些 HTML 或模板的一部分。

```html
<!-- Shows content only if a user is signed in... -->
<xf:if is="$xf.visitor.user_id">
	<!-- 做點什麼... -->
</xf:if>
```

if 標籤具有以下屬性：

- `is` - 滿足條件時應顯示標籤內容的條件。

#### 條件

該 `is` 屬性支援一些邏輯運算符：

- `OR` - 用於連接替代條件。（二擇一： `||`）
- `AND` - 用於連接其他條件。 (二擇一: `&&`）
- `!` - 放在條件前，使其反轉。（稱為："否"）
- `XOR` - 如果兩個條件中只有一個為 true，則返回 true。（稱為："異或"）

### Else/Else-If 標籤

else 和 else-if 標籤與 if 標籤結合使用，以按名稱建議的方式有條件地顯示 HTML。

**else 用法示例：**

```html
<xf:if is="$xf.visitor.is_admin">
	<!-- 這裡的內容只有管理員才會看到... -->
<xf:else />
    <!-- 這裡的內容將被顯示給任何非管理員的人看到！ -->
</xf:if>
```

**else-if 的用法示例：**

```html
<xf:if is="$xf.visitor.is_admin">
	<!-- 這裡的內容只有管理員才會看到... -->
<xf:elseif is="$xf.visitor.is_moderator" />
    <!-- 這裡的內容只會顯示給版主(不包括兼任管理員的用戶)。 -->
<xf:else />
    <!-- 這裡的內容將顯示給非管理員或版主的任何人。 -->
</xf:if>
```

正如你所看到的，一旦一個條件被滿足，其餘的 if 語句將被忽略。（所以，在這種情況下，如果使用者是管理員，上面的 `xf:if` 部分被執行，其餘的 if 語句則被忽略。)

### For-each 標籤

for-each 標籤使您可以遍歷項目陣列，為每個項目印出 HTML 區塊。

```html
<xf:set var="$names" value="{{ ['派翠克', '特麗莎', '金博爾', '韋恩', '格雷斯'] }}" />

<xf:foreach loop="$names" key="$key" value="$name" i="$i">
	<p>你好， {$name}。 這是名字 {$i} 。 該元素的陣列 key 值為： {$key}</p>
</xf:foreach>
```

for-each 標籤具有以下屬性：

- `loop` - 要循環的陣列。
- `key` - 在循環中使用的變數名稱，以獲取當前元素的陣列 key。 可以是整數（普通陣列）或字串（關聯陣列）。
- `value` - 在循環中使用的變數名稱，包含當前陣列項。
- `i` -  在循環中用於當前索引的變數名稱。

#### 輸出示例

> 你好，派翠克。 這是名字 1。
>
> 你好，特麗莎。 這是名字 2。
>
> 你好，金博爾。 這是名字 3。
>
> 你好，韋恩。 這是名字 4。
>
> 你好，格雷斯。 這是名字 5。

## 模板標籤

### 頭像標籤

在頁面中插入用戶的頭像。

```html
<xf:avatar user="{$xf.visitor}" size="o" canonical="true" />
```

頭像標籤具有以下屬性：

-   `user` - 要產生頭像的 XenForo 用戶物件。
-   `size` - 要產生的圖片大小。(參見圖片尺寸)
-   `canonical` - 是否使用有利於 SEO 的完整 URL。該值只對 `custom` 頭像有效。
-   `notooltip` - 當滑鼠懸停在頭像上時顯示的工具提示是否應該被關閉。
-   `forcetype` - 可以用來強制取得 `gravatar` 或 `custom` 頭像，設定值為其中之一。
-   `defaultname` - 如果 `user` 屬性包含一個無效的使用者時，預設要使用的使用者名稱。

#### 圖片大小

如果提供的頭像大小無效，則代碼將回退到大小 '`s`'。

-   `o` - `384px`
-   `h` - `384px`
-   `l` - `192px`
-   `m` - `96px`
-   `s` - `48px`

### 頁面路徑導覽列標籤

修改頁面路徑導覽列。
```html
<xf:breadcrumb href="{{ link('my_page') }}">{{ phrase('my_page_name') }}</xf:breadcrumb>
```
頁面路徑導覽列標籤具有以下屬性：

-   `href` - 為頁面路徑導覽列中的最後一個元素設定的連結。

標籤的值可用於設置頁面路徑導覽列中最後一個元素的名稱。

#### 其他用法
```html
<xf:breadcrumb source="$category.getBreadcrumbs(false)" />
```
您還可以通過 `source` 在頁面路徑導覽列標籤的屬性中呼叫函數來以程式設計方式定義自己的頁面路徑導覽列。

該 `source` 參數基本上採用一個具有 `href` 和 `value` 屬性的物件陣列（多維陣列），其中每個物件都是一個頁面路徑導覽列元素。

!!! note
	如果要更改根目錄頁面路徑導覽列，可以在 "基本信息面板" 選項部分中更改 "Root breadcrumb" 選項。

### 按鈕標籤

添加具有適當類的按鈕元素以及（可選）圖示。

```html
<xf:button icon="save"></xf:button>
```
button 標籤具有以下屬性：

-   `icon` - 應用於按鈕的圖示類。(參見按鈕圖示)

#### 按鈕圖示

默認情況下，XenForo 按鈕支援以下圖示（使用CSS創建）：

-   `add`
-   `confirm`
-   `write`
-   `import`
-   `export`
-   `download`
-   `disable`
-   `edit`
-   `save`
-   `reply`
-   `quote`
-   `purchase`
-   `payment`
-   `convert`
-   `search`
-   `sort`
-   `upload`
-   `attach`
-   `login`
-   `rate`
-   `config`
-   `refresh`
-   `translate`
-   `vote`
-   `result`
-   `history`
-   `cancel`
-   `preview`
-   `conversation`
-   `bolt`
-   `list`
-   `prev`
-   `next`
-   `markRead`
-   `notificationsOn`
-   `notificationsOff`
-   `merge`
-   `move`
-   `copy`
-   `approve`
-   `unapprove`
-   `delete`
-   `undelete`
-   `stick`
-   `unstick`
-   `lock`
-   `unlock`

### Callback 標籤

執行一個 PHP Callback 方法。
```html
<xf:callback class="Vendor\Addon\Class" method="getX" params="['abc']"></xf:callback>
```
此 callback 標籤具有以下屬性：

-   `class` - 該 class（從 root 命名空間）包含要執行的方法。
-   `method` - 要執行的方法。 (參見 callback 方法)
-   `params` - 提供給此方法的參數陣列。

#### Callback 方法

為了使一個方法被認為是 callback 方法，必須對其進行適當的命名，否則它將拋出錯誤 '`callback_method_x_does_not_appear_to_indicate_read_only`'。為了使它被認為是唯讀的，方法名稱必須以下列前綴之一開頭：

-   `are`
-   `can`
-   `count`
-   `data`
-   `display`
-   `does`
-   `exists`
-   `fetch`
-   `filter`
-   `find`
-   `get`
-   `has`
-   `is`
-   `pluck`
-   `print`
-   `render`
-   `return`
-   `show`
-   `total`
-   `validate`
-   `verify`
-   `view`

### CSS 標籤

加入一個外部 CSS 或 LESS 模板檔案。
```html
<xf:css src="mycss_file.css"  />
```
CSS 標籤具有以下屬性：

-   `src` - 要加入的 CSS 或 LESS 模板檔案。

#### 其他用法
```html
<xf:css>
html, body {
 font-family: "Roboto", sans-serif;
}
</xf:css>
```
如果 CSS 標籤不為空，則標籤中的所有內容都將轉換為嵌入式 CSS。

#### 進一步說明

對於[CSS]，不用再將它們稱為檔案了。將它們複製並貼上到新模板中，並按如下所示進行調用：<br>
來源自：[**克里斯·D，XenForo 開發人員**](https://xenforo.com/community/threads/including-external-library-js-and-css.136153/post-1185631)
```html
<xf:css src="questionthreads_fonticonpicker.css" />
<xf:css src="questionthreads_fonticonpicker_grey.css" />
```

### JS 標籤

加入一個JavaScript檔案。
```html
<xf:js src="myaddon/vendor/scripts/myjs_file.js"  />
```
JS 標籤具有以下屬性：

-   `src` - 要加入到模板中的 JS 檔案。
-   `prod` - 要加入到模板中的 JS 檔案，只適用於生產模式。
-   `dev` - 要加入到模板中的 JS 檔案，只適用於開發模式。
-   `min` - 是否要加入最小化版本的檔案。(用 `.min.js` 取代 `.js`)   -   只需要在生產模式下關心。
-   `addon` - 是否要使用開發中 JS 的 URL。  -   只需要在開發模式下關心。

!!! warning
	The `src` tag cannot be used in conjunction with either the `prod` or `dev` tags.

#### 其他用法
```html
<xf:js>
alert("真相很傷人，我知道。其實這是基於生物學上的。");
</xf:js>
```
如果 JS 標籤不為空，則標籤中的所有內容都會轉換為嵌入式 JS。

#### 進一步說明

JavaScript 檔案是相對於 `/js` 目錄提供的。儘管不建議這樣做，但您也可以在此標籤中加入外部資源。

這個標籤在 `editor` 模板中就是一個很好的例子。

### Set 標籤

set 標籤允許您建立一個對另一個變數的參考或建立一個新的變數。你應該在程式語言中使用變數的任何地方使用 set 標籤。
```html
<xf:set var="$visitor" value="{$xf.visitor}" />
```

!!! warning
	不要對要在多個模板中使用的一組元素使用 Set 標籤，而應使用 Macro 標籤。

!!! warning
	變數名稱（`var` 屬性）必須以 ` $` 開頭。

set 標籤具有以下屬性：:

-   `var` - 您想定義變數的名稱（本質上是別名）。
-   `value` - 要參考的變數或變數值。

#### 其他用法
```html
<xf:set var="$myVariableName">
我的變數值!
這可以是一個 callback，或者只是一組短語。
</xf:set>
```
如果 `value` 未提供屬性且標籤不為空，則變數值將設置為標籤的內容。

!!! warning
	當您以這種形式使用 Set 標籤時，該值將被轉義，並且結果值將是一個字符串。該 `value` 屬性雖然不支援 HTML 或類似 HTML 的標籤，但沒有此限制。

### Likes 標籤

顯示帖子的喜歡次數以及一些喜歡該帖子的用戶。

```html
<xf:likes content="{$post}" url="" />
```

likes 標籤具有以下屬性：

- `content` - 要顯示 '喜歡' 文本的 `XF\Entity\Post` 或 `XF\Entity\ProfilePost` 資料實體。
- `url` - 點擊 '喜歡' 文本時顯示的 URL。

#### 格式

> 你, 里斯本, kcho 和 2 其他人

格式為 [👍 `abc` 和 2 其他人]（其中 👍 "大拇指" 表情符代表 "點讚" 圖示，`abc` 代表最後三個點讚該帖子的使用者名字。）

### Sidebar 標籤

參見 [Sectioned 標籤](#sectioned).

### SideNav 標籤

參見 [Sectioned 標籤](#sectioned).

### Title 標籤

設定頁面的標題，包括頁面上的 `h1` 標籤和瀏覽器標籤。
```html
<xf:title>{{ phrase('my_page_title') }}</xf:title>
```
#### 進一步說明

雖然可以對 Title 進行永久性編輯，但是 **強烈建議** 您使用一個短語，以實現國際化，並增加網站管理員端的可定製性。

### Widget 標籤

在頁面中包含小元件，或將小元件添加到小元件的位置上。
```html
<xf:widget key="widget_name" />
```
widget 標籤具有以下屬性：

-   `key` - 小元件的 key，如小元件設定中定義的。
-   `position` - 如果設定，則改變小元件將被渲染的位置。
-   `class` - 不要與 HTML 的 class 混淆，這是包含小元件定義的 PHP 類。
	- `title` - 當使用 `class` 屬性時，您可以使用 `title` 屬性來設置小元件的標題。
	- 當使用`class`屬性時，你也可以提供小元件特定的選項作為屬性。

!!! warning
	`class` 標籤不能與 `key` 標籤一起使用。

### UserActivity 標籤

根據用戶上一次操作以及該操作發生的時間來顯示用戶的狀態。
```html
<xf:useractivity user="{$xf.visitor}" />
```
UserActivity 標籤具有以下屬性：

-   `user` - 用戶顯示的狀態。

#### 格式

> 查看頁面 _最新案例檔案_ · 4分鐘前

格式為 **[活動名稱]**  **· [時間]**

### UserBanners 標籤

在水平列表中顯示使用者的橫幅。

```html
<xf:userbanners user="{$xf.visitor}" />
```
UserBanners 標籤具有以下屬性：

-   `user` - 顯示該用戶的用戶橫幅。

#### 示例

![UserBanners 標籤的示例結果。](/files/images/example-userbanners-tag.png)

UserBanners 標籤的示例結果。

### UserBlurb tag

顯示用戶個人資料的單行摘要。
```html
<xf:userblurb user="${xf.visitor}" />
```
UserBlurb 標籤具有以下屬性：

-   `user` - 顯示 XenForo 使用者對象的簡介。

#### 格式

> FBI 顧問 · 43 · 來自美國

格式為 **[角色 / 自定義標題] · 年齡 · 位置**

### Username 標籤

Displays the user's username, optionally with a tool-tip.
顯示用戶的用戶名，可選擇使用工具提示。
顯示用戶的用戶名（可選）和工具提示。
```html
<xf:username user="{$xf.visitor.username}" notooltip="true" />
```
Username 標籤具有以下屬性：

-   `user` - 要顯示其名稱的 XenForo 用戶對象。
-   `notooltip` - 是否關閉工具提示。
-   `href` - 點擊用戶名時所導向到的連結。

!!! warning
	如果設定了 `href`，則不會顯示工具提示，因為它不起作用，而且可能會誤導使用者。

### UserTitle 標籤

Displays the user's title.顯示使用者的頭銜。
```html
<xf:usertitle user="{$xf.visitor}" />
```
UserTitle 標籤具有以下屬性：

-   `user` - 此 XenForo 用戶物件，用於顯示用戶頭銜。

### Sectioned 標籤

Sectioned 標籤全都會呼叫函數 `modifySectionedHtml`。 它們修改的 HTML 元素就是標籤名稱。因此 `sidebar` 標籤會修改側邊欄的 HTML 等等。

#### 示例
```html
<xf:sidebar>
 <h1>我的神奇側邊欄！</h1>
</xf:sidebar>
```
#### 共同屬性

-   `mode` - 修改模式。（請參閱修改模式）

#### 修改方式

預設情況下，修改方式為 `replace`。（即，如果沒有指定屬性。）

- `prepend` - 將標籤的內容放在 HTML 元素的開頭。
- `append` - 將標籤的內容放在 HTML 元素的末尾。
- `replace` - 用標籤的內容替換 HTML 的元素。

