# 依照需求創建映像檔--熟悉dockerfile

---

*複習：一個完整的dockerfile，裡面包含了做成一個映像檔會需要經過的所有步驟。*

Docker的映像檔是由一個基礎映像檔（base image）和一些環境設定所組成。
細部而言，裡面包含了作業系統(OS)、應用程式版本相依性(dependencies)、所有配置(configurations)。

最重要的特性有二：

 - 如果在某個docker環境下可順利運作，那麼在其他系統的docker環境下也都可以順利運作。
 - 有「基礎映像檔」的概念，也就是「使用輪子，而非重造輪子」的概念。
 => 製作映像檔的重點在客製化成自己所需的映像檔的過程。

這一節我們會詳細的探討如何寫一個完整的dockerfile。

#### 基礎映像檔
基礎映像檔也是可以`run`成容器的映像檔。
若要「定義」某個映像檔作為基礎映像檔，可以在dockerfile仿造以下範例指令：
```
FROM nginx:1.11-alpine
```

#### 配置（環境設定）
定義好基礎映像檔之後之後，就需要做配置的部分。
這四個指令會是最常用到的：`COPY`、`RUN`、`EXPOSE`、`CMD`。

 - `COPY <src> <dest>`
 複製本地端的src（是個檔案，寫路徑時為dockerfile所在目錄的相對路徑）到容器中的dest（是個檔案）。  
 以下是一個範例指令：  
 `COPY index.html /usr/share/nginx/html/index.html`  
 它將與dockerfile同樣目錄底下名為"index.html"的檔案複製到"/usr/share/nginx/html/index.html"這個檔案路徑。  
 
 
 - `RUN <command>`
 *與在終端機執行指令一樣的下command方式。用來執行指令。此章不會用到。*

 - `EXPOSE <port>`
 回顧第一章，我們需要連接容器的port才能溝通。`run`映像檔成為容器時要開什麼port就是這裡決定的。
 有以下三種寫法：
 	1. `EXPOSE 80`
 	2. `EXPOSE 80 433`
 	3. `EXPOSE 7000-8000`

 - `CMD ["command", "-a", "arga-value", "-b", "argb-value"]`
 定義`run`映像檔成為容器時要預先執行什麼指令。  
 以下是一個範例指令：  
 `CMD ["nginx", "-g", "daemon off;"]`  
 **註：這是「運行nginx」的指令：`nginx -g daemon off;`。*

<br/>
此章要製作的dockerfile內容如下：  
```
FROM nginx:1.11-alpine
COPY index.html /usr/share/nginx/html/index.html
EXPOSE 80
CMD ["nginx", "-g", "daemon off;"]
```

利用`build`生成名為"my-nginx-image"且TAG為"latest"的映像檔：
```
docker build -t my-nginx-image:latest .
```

接下來`run`起來：
```
docker run -d -p 80:80 my-nginx-image:latest
```

可用`curl`得到response：
```
curl -i http://docker  （用這個指令會得到完整的http response資料）
```
**註：如果說使用`curl docker`，就只會有網頁原始碼。*