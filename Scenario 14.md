
# Load Balancing Containers
**重要章節**

先備知識：
1. 完整了解load balancing的第一步就是了解微服務的**Service Discovery**觀念。這篇文章清楚明瞭：http://columns.chicken-house.net/2017/12/31/microservice9-servicediscovery/  
Service Discovery簡而言之就是「如何找到想找的容器所在位置」，這對容器的reconfiguring、scaling、錯誤容忍度等處理扮演著最關鍵的角色之一。
2. **NGINX Proxy**: 將外界對hostname發送的request分配到有能力處理此request的容器，以達到分擔loading。如何運行一個NGINX Proxy是這一個Scenario的重點。

## Step 1 - NGINX Proxy
運行一個NGINX proxy容器有三個重要步驟：
1. 使用port 80：
`-p 80:80`
2. 掛載`/var/run/docker.sock` 檔案，使得這個NGINX Proxy容器可以和docker daemon溝通。（對於/var/run/docker.sock的解說可參考此篇文章：https://blog.fundebug.com/2017/04/17/about-docker-sock/ ）
`-v /var/run/docker.sock:/tmp/docker.sock:ro` (<--read only)
3. 設定NGINX的預設host。我們在設定NGINX的config檔案時會清楚寫明哪一個host來的請求會由哪一台伺服器做處理，但為了預防沒有帶host的情況，NGINX預設會對這樣的請求回傳`HTTP status code=444`以關閉請求。如果說我們還是想處理沒有帶host的請求，就可以設定預設host，然後再寫明這個host該由哪一台伺服器做請求的處理。（解說可參考此篇文章：http://bit.ly/2GVwB3R ）
`-e DEFAULT_HOST=<domain>`

我們使用`jwilder/nginx-proxy`這個映像檔。組起來的運行指令如下：
```
docker run -d -p 80:80 -e DEFAULT_HOST=proxy.example -v /var/run/docker.sock:/tmp/docker.sock:ro --name nginx jwilder/nginx-proxy
```

當我們下curl指令（`curl http://docker`）時，會發現回傳`HTTP status code=503`，這是因為我們並還沒有任何其他容器被運行起來。

## Step 2 - Single Host

現在要來運行一個http-server，做的事情是在得到請求後回傳「處理請求的http-server」的機器名。範例中要使用的映像檔`katacoda/docker-http-server` 可做到這件事情。

我們利用以下指令將一個http-server運行起來：
```
docker run -d -p 80 -e VIRTUAL_HOST=proxy.example katacoda/docker-http-server
```
> 設定環境變數`VIRTUAL_HOST` 是為了要和NGINX Proxy所設定的default-host對應，使得沒有帶host的請求會被這一台http-server處理。

利用`curl http://docker` 即可看到結果。

## Step 3 - Cluster

如果說今天再運行一台同樣帶有`VIRTUAL_HOST=proxy.example`設定的容器，NGINX Proxy會依照`Round Robin`原則去把請求安排給這兩個http-server。意即：第一個請求交給第一個http-server、第二個請求交給第二個http-server、第三個請求交給第一個http-server，以此類推。

運行第二個（甚至是多個）http-server容器後，利用`curl http://docker` 即可看到請求都是輪流（`Round Robin`）被運行的機器運作。

## Step 4 - Generated NGINX Configuration

當一個新的、相同設定的http-server運行起來，NGINX Proxy其實會重建並重新load Config檔案。

若想看證據可輸入`docker logs nginx`，會看到以下log訊息：
```
dockergen.1 | 2018/03/04 11:00:09 Received event start for container e2c70b56d852
dockergen.1 | 2018/03/04 11:00:09 Generated '/etc/nginx/conf.d/default.conf' from 4 containers
dockergen.1 | 2018/03/04 11:00:09 Running 'nginx -s reload'
```

另外，我們用`docker exec nginx cat /etc/nginx/conf.d/default.conf` 可以看config檔案。較需要關注的內容如下：
```
# proxy.example
upstream proxy.example {
                ## Can be connect with "bridge" network
        # romantic_cori
        server 172.18.0.5:80;
                ## Can be connect with "bridge" network
        # gracious_mcclintock
        server 172.18.0.4:80;
----------------下面這段會在運行新http-server容器時出現----------------        
                ## Can be connect with "bridge" network
        # eager_noether
        server 172.18.0.3:80;
----------------上面這段會在運行新http-server容器時出現----------------
}
server {
        server_name proxy.example;
        listen 80 default_server;
        access_log /var/log/nginx/access.log vhost;
        location / {
                proxy_pass http://proxy.example;
        }
}
```