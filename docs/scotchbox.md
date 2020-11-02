# 附錄: Scotch Box

下面介紹如何將優秀的 [Scotch Box](https://box.scotch.io/) 安裝到自己的電腦上，以便在幾分鐘內通過簡單的命令就能擁有一個完全可操作的 XenForo 開發環境。

XenForo 有自定義的 Scotch Box 配置，它提供了執行 XenForo 所需的一切，包括偵錯程式和效能增強的資料快取。

Scotch Box 的運行環境在 [VirtualBox](https://www.virtualbox.org/) / [Vagrant](https://www.vagrantup.com/) 中。

## 安裝 Scotch Box 虛擬機

首先確定您希望虛擬 Web 伺服器在電腦上儲存其檔案的位置。建議您在自己的用戶主目錄中選擇一個位置。

在以下示例位置，我們將使用名為 *MyServer* 的目錄，此目錄位於您自己使用者目錄的根目錄中，並由您的名稱 *{username}* 標識：

- `/Users/{username}/MyServer` （Mac）
- `C:\Users\{username}\MyServer` （Windows）
- `/home/{username}/MyServer` （某些 Linux 發行版本）
- `/users/{username}/MyServer` （其他 Linux 發行版本）

選擇位置後，請按照以下步驟操作：

1. 在電腦上安裝 [VirtualBox](https://www.virtualbox.org/)
1. 在電腦上安裝 [Vagrant](https://www.vagrantup.com/)
1. 使用 **git** 客戶端，clone `https://github.com/scotch-io/scotch-box` 到 *MyServer* 目錄中。在上面的 Mac 示例位置中使用命令行 Client 端，指令是：

	```
	git clone https://github.com/scotch-io/scotch-box /Users/{username}/MyServer
	```

1. clone 過程完成後，下載此自定義 **Vagrantfile** 並覆蓋在 \*/Users/{username}/MyServer/Vagrantfile 中創建的 Vagrantfile : [下載自定義 Vagrantfile](/files/scotchbox/Vagrantfile)。

1. 自定義 Vagrantfile 放置到位後，執行以下命令：

	```
	cd /Users/{username}/MyServer
	vagrant up
	```

您的 Scotch Box 虛擬機現在已經創建並可以使用。

!!!Note
	Scotch Box 還提供了一個 '[Scotch Box Pro](https://box.scotch.io/pro/)' 版本的虛擬機，購買價格合理。如果你比較偏向喜歡使用 Scotch Box Pro，請參閱下面的部分 [介紹 Scotch Box 和 Scotch Box Pro 在配置和執行之間的差異](#scotch-box-pro)。

## 檔案都在哪兒？

一旦您的 Scotch Box 啟動並執行後，您可以將 XenForo 的 PHP 和 JS 檔案保留在您的主機上，從而允許您使用您所選擇的文字編輯器或 IDE，而虛擬機則負責通過其 Web 伺服器編譯和提供這些檔案。

您將能夠通過以下地址在 Web 瀏覽器中訪問新的 Web 伺服器：

 ```
 http://192.168.33.10
 ```
 
 Web 伺服器將從以下位置提取要提供的檔案

 ```
 /Users/{username}/MyServer/public
 ```
 
 如果你希望 XenForo 安裝在 `http://192.168.33.10/xenforo`，你應該把 XenForo 包中 `upload` 資料夾的內容放到 `/Users/{username}/MyServer/public/xenforo` 中。
 
## 停止和重新啟動伺服器
 
您可以在任何時候通過執行以下指令停止 Scotch Box 伺服器

```
cd /Users/{username}/MyServer
vagrant halt
```

... ，並且您可以通過執行重新啟動它

```
cd /Users/{username}/MyServer
vagrant up

```

!!!Note
	雖然 Vagrant / Scotch Box 會在您重新啟動電腦時自動關閉，但它不會自動再次啟動。

	每當您重新啟動時，您需要再次執行 vagrant up 指令才能使用伺服器。

## 官方文件

本指南摘自 Scotch Box 官方文件，該文件位於 <https://box.scotch.io>

## Scotch Box Pro

基本的 Scotch Box 需要一些額外的配置（通過自定義的 Vagrantfile 來傳遞）才能執行 XenForo 2，而 [Scotch Box Pro](https://box.scotch.io/pro/) 不需要額外的配置，無需下載額外的包就可以執行 XenForo 2。

要執行 Scotch Box Pro，請從 Scotch Box Pro 網站購買它，在您購買後將會收到一部分執行 *git clone* 指令的指示說明。

現在您可以使用與上述相同的說明進行安裝，唯一的例外是您應該下載 [此自定義的 Vagrantfile](/files/scotchboxpro/Vagrantfile)，而不是 Scotch Box 說明中列出的那個內容。