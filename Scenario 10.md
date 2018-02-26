
# Persisting Data Using Volumes

複習：
在Scenario 01有提過使用 `-v <host-dir>:<container-dir>` 使得`host-dir` 的資料和`container-dir` 可以同步（host資料能被容器存取、容器內對`container-dir`的改動都會同步影響`host-dir`）。  
在Scenario 07有提過data container的觀念，以及利用`--volume-from` 將data container的volume掛載(mount)而成為一個（或多個）容器的volume。

## Step 1 - Data Volumes
我們讓一個redis容器作為永久資料儲存區：
```
docker run -v /docker/redis-data:/data --name r1 -d redis redis-server --appendonly yes
```
> 1. 背景運行redis容器、把本地端`/docker/redis-data`目錄 map到redis的`/data`目錄 、取名r1。  
> 2. 執行`redis-server` 指令，將`appendonly`設為yes。（在redis容器的`/data`目錄會新建一個名為`appendonly.aof`的檔案。）

利用以下指令可以把當前目錄名為`data` 的檔案內容pipe到redis中：
```
cat data | docker exec -i r1 redis-cli --pipe
```
> 其實就是把檔案內容寫進`appendonly.aof`的檔案中。但redis裡面的檔案寫法和普通檔案不同，故如果看`/docker/redis-data/appendonly.aof`會看到不太一樣的內容。

## Step 2 - Shared Volumes
作法就和Scenario 07提到的`--volume-from` 作法相同。範例把redis的volume 掛載到ubuntu上，故下`ls /data` 指令一定可看到`appendonly.aof` 檔案。
```
docker run --volumes-from r1 -it ubuntu ls /data
```

## Step 3 - Read-only Volumes
Step1提到的`/docker/redis-data` 目錄也可以讓別的、新的容器做使用。用`-v` option就可辦到：
```
docker run  -v /docker/redis-data:/backup ubuntu ls /backup
```
> 1. 運行ubuntu容器、把本地端`/docker/redis-data`目錄 map到ubuntu的`/backup`目錄。
> 2. 在ubuntu執行`ls /backup`。（會有`appendonly.aof`這個檔案。）

用`-v`做掛載的時候會把所有權限原封不動給容器，如果說只想讓容器有部分功能（例如：只有讀取功能），可以參考以下指令的設定：
```
docker run  -v /docker/redis-data:/backup:ro ubuntu ls /backup
```
> `:ro` 代表容器內使用該目錄時是唯讀狀態的。