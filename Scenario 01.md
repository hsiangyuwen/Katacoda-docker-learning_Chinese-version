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
以下指令能達到上述的目標：
```
docker run -d redis:latest
```

以下指令可以列出所有正在運行的容器，以及它們的ID(container-id)、映像檔名(repository:tag)、名字(friendly-name)：
```
docker ps
```

以下指令可以列出指定的容器的更詳細資訊，例如IP位址：
```
docker inspect <friendly-name|container-id>
```

以下指令可以秀出所有output內容（被寫進stderr或stdout的內容）。
```
docker logs <friendly-name|container-id>
```
＊上述的friendly-name就是```docker ps```所顯示的name，剛剛在做```docker run```的時候並沒有指定名字，所以docker會自動幫我們創建一個。


## 存取（Access）容器

我們將容器運行起來，但容器仍舊屬於無法和外界溝通的情況，等於只是一個在主機上運行、與外界無聯繫的盒子。如果容器外面的process想要access這個容器，就需要這個容器有開通一個port讓外面的process可以連接。
Jane發現使用以下指令便可以在運行容器時，加入以下option便可以和容器的port做連接：
```
-p <host-port>:<container-port>
```
Jane現在要做的事情是為新建的Redis容器取名，並且讓這個容器的某個port開通，並且和這個port做連接。
使用以下範例指令便可以一次完成這三件事：
```
docker run -d --name redisHostPort -p 6379:6379 redis:latest
```
這個指令做了以下幾件事情：
- 運行redis:latest這個映像檔成為一個容器
- 將這個容器的friendly-name取為"redisHostPort"
- 將主機的6379這個port，和容器的6379這個port做連接（作為溝通橋樑的兩端）
- 在背景中執行

另外，也可以指定IP位址：
```
-p 127.0.0.1:6379:6379
```

如果今天沒有指定```host-port```，主機會隨便指定一個可以使用的port去和container的port做連接。
而這時會不知道主機使用的port是哪一個，但我們可以經由以下指令看到：
```
docker port redis_name 6379
```

## 讓容器內資料永久存在不消失

幾天後，Jane發現每當她刪除並重新創建容器時，裡面曾經存的資料都會消失。這並不是她樂見的事情。將主機的某個目錄與容器內某個目錄做連接，便可以解決這個問題。
以下是在運行容器的同時將某個主機的目錄與某個容器的目錄做連接所需要多設定的option：
```
-v <host-dir>:<container-dir>
```

以下是一個範例指令：
```
docker run -d --name redisMapped -v /opt/docker/data/redis:/data redis:latest
```
這個指令做了以下幾件事情：
- 運行redis:latest這個映像檔成為一個容器
- 將這個容器的friendly-name取為"redisMapped"
- 將主機的"opt/docker/data/redis"與容器的"/data"做連接
- 在背景執行這個容器

這個設定的結果是，當容器對這個目錄內資料做新增、編輯、移除等變動，主機內的那個目錄也會同步做動作。（兩者下```ls```指令時所看到的內容必定相同）


## 使用容器的shell介面

如果沒有在運行時下```-d```的option，容器就會在前端執行。
另外，如果想要進入容器使用它的shell，會用'''-it'''這個option。以下是一個範例指令：
```
docker run -it ubuntu bash
```
＊當使用完畢退出shell介面(exit)時，會發現整個容器也會跟著被停止。
