# Gitbook 使用說明

- 先 clone 這個 repo 到本地。如果 repo 已經存在那就 `git pull` 一下。

- 直接編輯現在 boilerplate 裡面的檔案。如果想製造出新的章節，就去 `SUMMARY.md` 裡面增加。增加了以後運行`gitbook init`。`gitbook`發現新的章節檔案不存在，會幫你製造出新的空檔案。

- 可以一邊編輯一邊在 `localhost` 預覽：`gitbook serve`。接著瀏覽器打開 [http://localhost:4000](http://localhost:4000/) 就能預覽。不想用 4000 的話也可以指定 port： `gitbook serve --port 5566`

- 要發布前：**先確定已經 commit 所有 changes 到 master**。然後跑 `./publish.sh` 這隻 script，他會自動幫你編譯然後把檔案 commit 到 gh-pages 這個分支，推到 GitHub 上面，觸發自動部署。你應該可以在 [https://vinhuang0228.github.com/notes](https://vinhuang0228.github.com/notes) 看到你的 GitBook。

- 可以生成各種格式的電子書：`gitbook pdf ./ ./mybook.pdf` 或 `gitbook epub ./ ./mybook.epub` 或 `gitbook mobi ./ ./mybook.mobi`

- Other question: 先 git init , 然後看`git add .` `git commit -m "新加的內容是什麼"`
