
# Docker Metadata & Labels

Label的功能在把docker架構的開發產物產品化的時候非常容易使用到，它可以用來區分產品名稱、運行版本、擁有者、使用的伺服器...等「資訊(metadata)」。

Docker有提出關於使用label的建議規則：
1. 要有**reverse domain name notation**作為label的前綴。詳細可參考這篇wiki：https://en.wikipedia.org/wiki/Reverse_domain_name_notation
2. 如果較常被CLI使用，為了打字上的方便，會使用**domain name notation**作為label的前綴。
3. 只能包含`小寫a-z`、`0-9`、`.`、`-` 

## Step 1 - Labels for Docker Containers
一個container可以有多個label。在`docker run`的時候使用`-l`這個option即可。範例如下：
```
docker run -l user=12345 -d redis
```

如果要用「檔案」的方式存label，會使用`--label-file`這個option：
```
echo 'user=123451234' >> labels && \
echo 'role=cache' >> labels

docker run --label-file=labels -d redis
```

## Step 2 - Labels for Docker Images
（註：image label和image tag是完全不同的東西。）
一個image可以有多個label。在`Dockerfile`裡面使用`LABEL`這個instruction，然後做`docker build`即可：
```
LABEL owner=katacoda
```

如果要加入多個label，要使用右斜線（`\`）隔開：
```
LABEL vendor=Katacoda \ com.katacoda.version=0.0.5 \ com.katacoda.course=Docker
```

## Step 3 - Inspecting Labels
容器：
```
docker inspect -f "{{json .Config.Labels }}" <container-name|id>
```

映像檔：
```
docker inspect -f "{{json .ContainerConfig.Labels }}" <image-name|id>
```

## Step 4 - Query By Label
查找容器：
```
docker ps --filter "label=user=scrapbook"
```

查找映像檔：
```
docker images --filter "label=vendor=Katacoda"
```

註：查找是會區分大小寫的，因此非常建議遵守命名的建議規則。

## Step 5 - Labels for Docker Daemon
***(可略)***
語法：
```
docker -d \
-H unix:///var/run/docker.sock \
--label com.katacoda.environment="production" \
--label com.katacoda.storage="ssd"
```