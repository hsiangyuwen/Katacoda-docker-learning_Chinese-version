
# Docker Networks & Embedded DNS Server

Docker Networks是另一個把不同容器做連結的方式。Docker Networks與現今的網路架構比較相像，並且也是Docker較為推薦的容器連結解決方案。  
（從另一個角度來看，`--link`的方式會被淘汰掉。目前也已經被Docker證實。目前`--link`比較不能被捨棄的原因在於環境變數共享的設定，這是Docker Network沒有的，但當然會有基於Docker Networks底下的解決方案。詳細可以參考 **Legacy container links** 這篇docker document：https://docs.docker.com/network/links/ ）

## Step 1 - Create Network
建一個新的network的範例指令如下：
```
docker network create backend-network
```
建好之後，我們可以在運行新容器時加上`--net` 這個option以加入這個network。範例指令如下：
```
docker run -d --name=redis --net=backend-network redis
```

## Step 2 - Network communication
和Scenario 08提到的`--link`不同的是，接上network不會設定環境變數，也不會在/etc/hosts加上新的entry。最主要是藉由Embedded DNS Server運作。  
每個容器都會有個Embedded DNS Server，IP設在127.0.0.11。設定檔則在`/etc/resolv.conf`。  
（`resolv.conf` 介紹可參考wiki：https://en.wikipedia.org/wiki/Resolv.conf ）

我們可以想像，在backend-network中，已加入此network的redis容器有一組ip位置，而DNS domain為 `redis` 。如果說想要知道確切的ip，我們可以直接ping redis容器，方法如下：
```
docker run --net=backend-network alpine ping -c1 redis
```
> 運行一個alpine容器去ping redis容器。  
> `-c1` 代表ping一次

## Step 3 - Connect Two Containers
現在我們要讓Node.js web app和一個redis做連結（和Scenario 08所做的事情一模一樣，但現在是使用Docker Network去辦到。），步驟如下：
1. 建一個新的network：  
`docker network create frontend-network`
2. 將「已經運行的」redis容器連上這個新network。要使用`docker network connect`：
`docker network connect frontend-network redis`
3. 運行一個Node.js web app，並加入這個新network。要使用`--net` option：
`docker run -d -p 3000:3000 --net=frontend-network katacoda/redis-node-docker-example`

## Step 4 - Create Aliases
我們也可以讓一個容器在連上network時，幫他取在這個network內的別名。這和link取別名是一樣的概念。  
取別名是在使用`docker network connect`連線時多加`--alias`這個option。
範例如下：
1. 建一個新的network：
`docker network create frontend-network`
2. 將「已經運行的」redis容器連上這個新network，並取別名為`db`：
`docker network connect --alias db frontend-network2 redis`
3. 運行一個alpine容器，並加入這個新network。然後去ping `db`這個別名（也就等同ping redis容器）。
`docker run --net=frontend-network2 alpine ping -c1 db`

## Step 5 - List Networks & Disconnect Containers
列出你的docker engine目前有的network：
```
docker network ls
```

如果要針對某個network看詳細資訊，和我們看容器的詳細資訊時一樣，都使用`inspect`：
```
docker network inspect frontend-network
```

如果要讓某個容器從某個network離開，會使用`disconnect`：
```
docker nerwork disconnect frontend-network redis
```