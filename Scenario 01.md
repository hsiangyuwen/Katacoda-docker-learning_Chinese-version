# [第 01 節] 部署你的第一個Docker容器

---

## 什麼是Docker？
Docker這個工具的定義是"An open platform for developers and sysadmins to build, ship, and run distributed applications."，意思就是能讓使用者快速部署與執行分散式應用程式的一個工具。

Docker最重要的就是Container（容器）的觀念。我們可以把容器想成是一個運行在我們主機上的一個process，運行著一個應用程式，並且擁有自己的dependency。而我們的主機上可以運行著很多個容器，各自獨立運作。

## 本節須知
在這一節當中，我們是一個想要為自己的應用程式部署Redis（一種非關聯式資料庫）的工程師－－Jane。本節會演示部署Redis容器的詳細步驟。
＊終端機已經有最新版本docker，可以用終端機操作。另外，主機名為docker。

## 找尋並運行容器
最一開始要先找到自己想要運行起來的image（映像檔）。映像檔好比是一個程式，運行起來就成為了程序，也就是容器。映像檔包含了所有用來運行所需的所有東西，故主機不會因為需要將這個映像檔運行起來而要去調整或是符合什麼dependency。
我們用以下指令去尋找名為Redis的容器：
```
docker search redis
```

Jane藉由搜尋知道了他想要運行起來的映像檔名為"redis"，並且想要運行最新的版本。另外，因為Redis是資料庫，Jane希望它能在背景中運行。
以下指令能達到以上三個目標：
```
docker run -d redis:latest
```

以下指令可以列出所有正在運行的容器，以及它們的ID、映像檔名、名字：
```
docker ps
```

以下指令可以列出指定的容器的更詳細資訊，例如IP位址：
```
docker inspect <friendly-name|container-id>
```

以下指令可以秀出所有output內容（被寫進stderr或stdout的內容）。
```
docker log <friendly-name|container-id>
```
＊上述的friendly-name就是```docker ps```所顯示的name，剛剛在做```docker run```的時候並沒有指定名字，所以docker會自動幫我們創建一個。


## 存取（Access）容器

