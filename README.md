# Gitbook使用說明

- 先 clone 這個 repo 到本地。如果 repo 已經存在那就 `git pull` 一下。

- 直接編輯現在boilerplate裡面的檔案。如果想製造出新的章節，就去 `SUMMARY.md` 裡面增加。增加了以後運行`gitbook init`。`gitbook`發現新的章節檔案不存在，會幫你製造出新的空檔案。

- 可以一邊編輯一邊在 `localhost` 預覽：`gitbook serve`。接著瀏覽器打開 [http://localhost:4000](http://localhost:4000/) 就能預覽。不想用 4000 的話也可以指定port： `gitbook serve --port 5566`

- 要發布前要編譯： `gitbook build`。預設是當前路徑，也可以指定路徑：`gitbook build [書籍路徑] [輸出路徑]`

- 可以生成各種格式的電子書：`gitbook pdf ./ ./mybook.pdf` 或`gitbook epub ./ ./mybook.epub`

或 `gitbook mobi ./ ./mybook.mobi`

