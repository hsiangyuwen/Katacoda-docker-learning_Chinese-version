
# Creating Connections Between Containers using Links

這一個Scenario會講述如何用 `--link` 讓兩個容器溝通，範例是想要讓一個web app和一個data container做連結，這是一個最常用的使用情境。

### Step 1 - Start Redis
首先，我們先創一個redis server。
```
docker run -d --name redis-server redis
```
> --name: 為運行起來的服務取一個清楚易懂的名稱(friendly name)。

### Step 2 - Create Link
如果要讓某個容器連上redis server，會在運行這個容器時加上 `--link` 這個option。範例code如下：
```
--link <container-name|id>:<alias>
```
> 1. **container name | id** : 想要連接上的容器的friendly name或是id。
> 2. **alias**: 為這個link關係中，想要連接上的那個容器取一個別名。簡單來個舉例：在一個複雜的容器運行環境中，可能會有很多個data container，例如redis1、redis2、...、redis???，但我們可以統一把所有link關係都叫做`redis` ，這個 `redis` 就是所謂的別名(alias)。在以前版本的 `docker ps` 指令會和你為新容器的取名一同顯示在NAMES欄位，現在已經看不見了。

link實際上會做以下兩件事情：
1. 基於你想link的容器，去對你想運行的新容器的環境變數做設定。可以用 `env` 這個指令去看所有的環境變數。
範例指令如下： `docker run --link redis-server:redis alpine env`
2. 會對你想運行的新容器的 `/etc/hosts` 檔案做更新，加上一個代表你想link的容器新的HOSTS entry。這個entry的值包含`alias`、`hash id`、`friendly name`：。我們可以用 `cat /etc/hosts` 印出所有的HOSTS entry，範例指令如下：
`docker run --link redis-server:redis alpine cat /etc/hosts`
> 這兩個範例指令並不會運行一個alpine容器。只有展示實際上運行時會發生的事情而已。

### Step 3 - Connect to Web App
接下來就來實際的啟用一個web app容器，並連接到最一開始運行的redis-server。
```
docker run -d -p 3000:3000 --link redis-server:redis katacoda/redis-node-docker-example
```

### Step4 - Connect to Redis CLI
範例指令如下：
```
docker run -it --link redis-server:RS redis redis-cli -h RS
```
> 拿redis這個映像檔運行起來一個容器，連接到redis-server，並且把redis-server取一個別名叫做RS。
> 利用 `-it` 進到容器的CLI介面
> 輸入 `redis-cli -h RS` 指令。`redis-cli` 是運行redis client的command、`-h` 是指定連到哪裡的option、RS是redis-server的alias。在這裡我們真正用上了alias，可以想像在複雜的容器運行環境中，alias是一個重要的功能。