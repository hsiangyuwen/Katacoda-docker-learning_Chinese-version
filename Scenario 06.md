
# Ignoring Files

---

在創建映像檔的時候大多都有利用 `COPY` 或是 `ADD` 複製整個working directory的情況，如果說裡面有比較機密的資料（例如：secret key、帳號密碼），當別人利用你創建的映像檔建立容器時，就會一起得到這些機密資料。  
在這樣的情況下，只要編輯一份 `.dockerignore` 檔案就可以在創建映像檔的時候忽略某些檔案。

假設今天在 `Dockerfile` 相同路徑下有一個文件叫做 `password.txt` ，只要在terminal內、Dockerfile路徑下打以下指令就可以創建需要的 `.dockerignore` 檔案：
```
echo passwords.txt >> .dockerignore
```
＊更複雜的 `.dockerignore` 寫法在docker文件中有詳細提到：https://docs.docker.com/engine/reference/builder/#dockerignore-file

＊如果說在創建映像檔的時候會需要用到這些機密資料，可以不在 `.dockerignore` 包含 `password.txt` ，而是在使用該檔案的同一個 `RUN` 指令下多加 `rm` 指令。  
範例： `RUN cat password.txt && rm password.txt`

除了機密資料以外，大而無用的檔案、 `.git` 資料夾、`node_modules/` 資料夾，都是可以在創建映像檔時忽略掉的檔案。

