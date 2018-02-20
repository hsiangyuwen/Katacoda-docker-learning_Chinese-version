
# OnBuild Instruction

---

我們在Scenario 04所完成的dockerfile如下，本章用「舊dockerfile」代稱：
```
FROM node:7-alpine
RUN mkdir -p /src/app
WORKDIR /src/app
COPY package.json /src/app/package.json
RUN npm install
COPY . /src/app
EXPOSE 3000
CMD [ "npm", "start" ]
```
在Senario 04的建立映像檔指令如下：
```
docker build -t my-nodejs-app .
```
如果說今天有新的dockerfile，要做的事情包含了所有舊dockerfiler要做的事，還多加了一些設定，那是不是要複製整份舊dockerfile呢？  
->不可能，一定會去繼承舊的映像檔，畢竟都有「基礎映像檔」的觀念了。

**--------------------以下做法會被糾正!--------------------**  
因為`package.json` 一定要使用新dockerfile路徑對應的那一份才行，所以舊dockerfile應該更新為：
```
FROM node:7-alpine
RUN mkdir -p /src/app
WORKDIR /src/app
```
而任何想去繼承舊dockerfile所做出之基礎映像檔的dockerfile，都該長以下這樣：
```
COPY package.json /src/app/package.json
RUN npm install
COPY . /src/app
...
CMD [ "npm", "start" ]
```
該這樣子進行嗎？明明「`COPY`, `RUN`, `COPY`」那三行不管何時都要做，好想讓他們包含在舊dockerfile當中啊！  
**--------------------以上做法會被糾正!--------------------**

這時就有`ONBUILD` 這個instruction誕生了。  
他會讓有加上`ONBUILD` 作為前綴的dockerfile指令在被繼承時才去執行。

所以舊dockerfile該被改成這樣子：
```
FROM node:7-alpine
RUN mkdir -p /src/app
WORKDIR /src/app
ONBUILD COPY package.json /src/app/package.json
ONBUILD RUN npm install
ONBUILD COPY . /src/app
CMD [ "npm", "start" ]
```
建立映像檔指令如下：
```
docker build -t node:7-onbuild .
```
新的dockerfile該長成這樣子：
```
FROM node:7-onbuild
EXPOSE 3000
```
### 小結
`ONBUILD` 幫助了以下三件事達成：

1. 舊dockerfile在`docker build` 的時候不會執行有`ONBUILD` 前綴的指令
2. 新的dockerfile在`docker build` 的時候會完整執行基礎影像檔的dockerfile指令（包含`ONBUILD` 的）以及自己的dockerfile包含的指令。
3. 有效簡短繼承舊dockerfile時需要寫的指令量。

*補充：祖父dockerfile中的`ONBUILD` 在孫子dockerfile做`docker build` 時不會執行。

*補充：http://www.macadamian.com/2017/03/14/docker-build-squash-vs-onbuild/  
這是`--squash` 參數，另一種優化`docker build` 速度的方式。
