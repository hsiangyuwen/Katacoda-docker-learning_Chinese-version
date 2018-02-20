
# Create Data Containers

---

在Scenario 01裡面有提到 `-v <host-path>:<container-path>` 是一個將主機的某個目錄與容器內某個目錄做連接的option，使得資料在容器重新create的時候，資料都不會消失。這也是data container必要的條件之一。

## 創建data container
首先，我們會先創建一個data container容器。我們使用 `docker create` 去創建，而不是直接 `docker run` 。指令如下：
```
docker create -v /config --name dataContainer busybox
```
這個指令做了以下幾件事情：
- 選用"busybox"這個映像檔創建容器
- 將`當前目錄`和容器內的 `/config` 資料夾做mapping。（也就是說兩邊目錄內的資料都會保持一致）

接下來我們要複製本地端的資料到容器內，我們使用 `docker cp` ：
```
docker cp config.conf dataContainer:/config/
```

## Mount Volume From
data container 還有一個用處是作為資料存放處，以便之後運行的容器可以輕鬆取用裡面的資料。這裡的觀念和linux系統中的mount相同。

做法是在運行新容器時，多加上 `--volume-from` 這個option。範例指令如下：
```
docker run --volumes-from dataContainer ubuntu ls /config
```
這個指令做了以下事情：
- 運行一個ubuntu容器
- 直接將`dataContainer` 這個容器的volume作為這個新容器的volume。
- 檢視`/config` 底下的檔案名稱

## Export/Import Container
不只是data container而已，其實container本身也可以直接包成一個像是映像檔的東西交付給別人。當然，曾經在這個容器有過的操作都會保留下來，和 `docker image` 是不一樣的。
這個動作的指令是`docker export` ，它會將容器轉成tar檔的形式。範例指令如下：
```
docker export dataContainer > dataContainer.tar
```
此步驟是將dataContainer輸出成tar檔，存在當前目錄。

如果要import容器，會使用`docker import` 。範例指令如下：
```
docker import dataContainer.tar
```
此步驟是將當前目錄裡的tar檔，變成"映像檔"。（可用`docker images`看到）