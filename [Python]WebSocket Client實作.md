# 前言
客戶的需求真是千奇百怪，今天遇到了需要將樹梅派(RaspberryPi)的GPIO訊號傳到Python應用程式做整合應用的需求，思考一下應該是要好好地挑選一個全雙工的網路通訊結構去套用，突然腦中閃過websocket，那個早在2011年就已經被標準化，直到HTML5支援Websocket開放原生方法，才逐漸在Web界被廣泛應用的websocket阿，於是就先透過nodejs架一個websocket server放在樹梅派，負責接收GPIO訊號並利用websocket廣播，然後python應用程式寫一個websocket client就可以接收拉，這篇就簡單介紹並筆記一下，怎麼實作python的websocket client，不難，套件而已。

# 安裝websocket-client for python套件
我們就是站在巨人的肩膀上寫系統，所以我們使用日本大神前輩的套件，先pip一下
```python
> pip install websocket-client
```

# 產生websocket的連線
直接將websocket的網址，傳入物件裡面，就會開始嘗試連線，這邊可以一起傳入各種事件觸發時候要執行的function，或是稍後在指定也是可以的
```python
from websocket import enableTrace, WebSocketApp

# 取物件的時候就直接指定事件方法
ws = WebSocketApp(
    "ws://localhost:9453",
    on_message=MessageFunc,
    on_error=ErrorFunc,
    on_close=CloseFunc
)

# 取完物件再指定事件方法
ws.on_open = OpenFunc
```
這幾種事件觸發的時機，分別如下：
- on_open：開啟連線成功時執行
- on_message：收到來自server端傳來的訊息時執行
- on_error：連線發生錯誤時執行
- on_close：關閉連線時執行

接著，啟動連線
```python
ws.run_forever()
```
最後，當websocket用完的時候，很重要的動作就是要關閉，以免佔用server端太多連線，造成server端的負荷
```python
ws.close()
ws = None
```

在筆者整合應用的例子中，我會在on_message的時候，等收到來自server端傳來，哪一個GPIO pin腳有反應，就同步顯示在python應用程式的畫面上，可以得知哪一個地方的電燈被打開，或是哪一扇門被打開，達到一種遠端管理與控制的作用，筆者覺得如果有這類單純訊號網路傳遞的需求，用websocket是最簡單便利的方式了。

另外，套件作者還有SSL與proxy機制的設計，筆者暫時沒有用到，有興趣的朋友可以參考他的github文件喔

https://github.com/websocket-client/websocket-client