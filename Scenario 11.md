
# Managing Log Files

docker 的log是選定的容器它stdout和stderr所吐出的資料所成的集合。  
聽起來簡單，但日誌(logging)和監視(monitoring)微服務是微服務非常重要的一環，開源、不開源的工具非常的多；作為解決系統問題的最主要資訊來源，這個領域是個非常值得深入學習的部分。

# Step 1 - Docker Logs
看log：
```
docker logs redis-server。
```
> 補充常用的option：
> 1. `-f` 或 `--follow` 可以追蹤即時的log。例如：`docker logs -f redis-server`
> 2. `--tail=100` 可以列出最新100筆的log。例如：`docker logs --tail=100 redis-server`

# Step 2 - SysLog
docker預設會把log存成一個json檔然後存於本地端，可以預見這樣的做法會讓本地端有一個存著log資訊的大檔案。其實docker除了預設的`json-file`這個「log driver」之外，支援了非常多其他log driver（可以想成很多log儲存方案），甚至是第三方log driver，這一節要介紹的是**SysLog**（它不是第三方的log driver）。  

開發者在對微服務實作logging功能的時候，常會想到使用ELK，也就是Logstash -> ElasticSearch -> Kibana，包辦了log的所有處理、查詢、呈現。在使用ELK時，若日誌是能從SysLog這個log driver得到，設定上會方便很多。

當然，Docker在1.12版本後開放使用多元的第三方log driver，可參考這篇文章：https://www.ccc.tc/notes/docker-logging-driver

下面指令能運行一個會log到syslog的redis容器，方法為設定`--log-driver`這個option：
```
docker run -d --name redis-syslog --log-driver=syslog redis
```

＊很重要的一點：只有使用`json-file` 作為log driver的容器，才能用`docker logs` 印出log。使用其他的log driver必須要

# Step 3 - Disable Logging
只要將`--log-driver`這個option設為none即可：
```
docker run -d --name redis-none --log-driver=none redis
```
---
這一節的最後有提到利用`docker inspect`去看log config，但沒有講述如何調整log configuration。調整的部分可以參考docker 文檔：https://docs.docker.com/config/containers/logging/configure/
