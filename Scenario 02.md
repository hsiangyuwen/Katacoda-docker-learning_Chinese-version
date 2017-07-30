# 初探Dockerfile

---

Docker的映像檔是由一個基礎的映像檔（base image）和其他的環境設定所組成。這些都可以用dockerfile去描述，寫好後再編譯那個dockerfile即完成映像檔的創建。

以下是一個範例：
```
FROM nginx:alpine
COPY . /usr/share/nginx/html
```
簡單解釋：
- 將nginx:alpine這個映像檔作為基礎映像檔
- 將主機當前目錄下的檔案複製到容器的那個目錄下。