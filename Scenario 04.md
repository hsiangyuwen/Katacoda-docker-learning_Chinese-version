
# 部署 Node.js web app 容器

---

### 基礎映像檔

此章節會使用到的基礎映像檔是Node.js的pre-build 映像檔：`node:7-alpine` 。它比起官方正式版的映像檔小，會更適合拿來作為基礎映像檔，再自己做參數設定。

我們在dockerfile裡面可以創建一個新的資料夾，並且把它作為working directory。指令如下：

```
RUN mkdir -p <path>
WORKDIR <directory>
```

`-p` 這個option是用來創建父目錄，以防子目錄創建時找不到父目錄會報錯的狀況。  
`WORKDIR` 則是指未來下的指令都以它作為相對路徑的起點。

依據我們的需求，dockerfile如下所示：
```
FROM node:7-alpine
RUN mkdir -p /src/app
WORKDIR /src/app
```

### 環境設定

環境設定這部分，對Node.js web app來說最重要的就是要從npm把所有需要的package裝好。一般來說會先建好一個`package.json` 檔案，然後做`npm install` 。所以我們在dockerfile要做的事情就是將`package.json` 複製進容器當中，然後下`npm install` 指令。
故dockerfile中要新增的指令如下：
```
COPY package.json /src/app/package.json
RUN npm install
```
*註：`COPY`還是要列完整的路徑，和`WORKDIR`無關。`RUN` 則就是會在我們的`WORKDIR` 做。*

這部分在預設情況下，docker會使用cache，也就是說如果`package.json`的內容沒變的話，就可以使用存於cache中的cache result，以加速以後的`RUN npm install`。  
當然，如果每次都想要`npm install`重裝，可以在`docker build`時多加一個option:  `--no-cache=true`。

### 複製程式碼與最後步驟

最後我們將source code複製到容器裡，以及做完開port，就完成了設定。指令如下：
```
COPY . /src/app
EXPOSE 3000
```

`docker run` 等於是把映像檔內（也可以說是dockerfile內）所指定要做的事情做完，所以就Node.js web app而言，一定還要有類似"start app"的動作。對Node.js來說是`npm start` ，因此dockerfile還要有這一行指令：
```
CMD ["npm", "start"]
```

大功告成，我們可以來建立映像檔了！
建立映像檔指令如下：
```
docker build -t my-nodejs-app .
```
建立容器指令如下：
```
docker run -d --name my-production-running-app -e NODE_ENV=production -p 3000:3000 my-nodejs-app
```
*註：`-e` 這個option會在之後章節解釋，他與設定容器環境變數有關。
