
# Ensuring Container Uptime With Restart Policies

容器也是會crash的，我們可以在運行它時就設定它在crash時該做的行為。

## Step 1 - Stop On Fail
預設容器在crash後就停止運行，因為它會是stop狀態，不是消失，故我們可以使用`docker ps -a`看到所有存在的容器的狀態。  
通常crash的容器會被標注`Exited (1) ? seconds ago` 等等的狀態。如果crash了，觀看這個container的log就變成了非常重要的一件事，就會用到Scenario 11所提到的方法。

## Step 2 - Restart On Fail
如果要讓容器在crash後會自動重啟，我們會在運行容器時多加`--restart`這個option。範例指令如下：
```
docker run -d --name restart-3 --restart=on-failure:3 scrapbook/docker-restart-example
```
> **on-failure**：意指crash的時候才會restart。
> **:3**：重啟三次(仍不斷crash)後就停止。所以一個會造成容器不斷crash的映像檔，會運行容器4次。

## Step 3 - Always Restart
將`--restart`這個option設成`always`即可：
```
docker run -d --name ALWAYS_RESTART --restart=always scrapbook/docker-restart-example
```