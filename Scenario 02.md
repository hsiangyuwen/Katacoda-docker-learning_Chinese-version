# 初探Dockerfile

---

Docker的映像檔是由一個基礎映像檔（base image）和一些環境設定所組成。這些都可以用dockerfile去描述，寫好後再編譯那個dockerfile即完成映像檔的創建。

以下是一個範例：
```
FROM nginx:alpine
COPY . /usr/share/nginx/html
```
簡單解釋：
- 將nginx:alpine這個映像檔作為基礎映像檔
- 將主機當前目錄下的檔案複製到容器的那個目錄下。

經由```build```指令，docker指令介面會依序執行dockerfile裡面的每一行指令，並且最後生成一個可以```run```的映像檔。

以下是```build```指令的範例指令：
```
docker build -t webserver-image:v1 .
```
上述指令中，```-t <repository:tag>```是要你對生成的這個映像檔取一個repository名稱（即image名稱），以及通常作為區分不同版本的TAG。
所以下達了這個指令，我們便建立了一個名為"webserver-image"，且有TAG為"v1"的映像檔，存到當前目錄。

如果要列出目前本機有的image，可用以下指令：
```
docker images
```

現在我們來```run```我們創建出來的映像檔：
```
docker run -d -p 80:80 webserver-image:v1
```
我們可以用```curl```這個指令去要網頁原始碼：
```
curl docker
```
