# 前言
原本採用web架構，但web對於RTSP支援程度較差(MJPG才是網頁常見的格式，原生支援度高)，目前使用的作法是透過架NodeJS接收RTSP串流，再經由websocket傳輸到瀏覽器畫面上，但這樣多一轉換層，除了效能的疑慮，也會有一些系統不穩定的風險，假如websocket斷掉，影像就進不來了，而且同一時間要連的RTSP不只有一個，不確定websocket是否能承受大量的資訊傳輸，因此覺得還是client直接接收RTSP串流比較穩一點；原本的想法是在react裡面埋nodejs，透過VNC抓RTSP的dll元件來去達成目的，但工程浩大，也還不知道怎麼做，最後整個重寫，使用python tkinter來呈現RTSP攝影機的影像了。

# 安裝opencv for python套件
這篇的重點就是要用OpenCV來讀RTSP，當然套件是不能少的，先pip一下吧
```python
> pip install opencv-python
```

# 接著讀取RTSP的影像串流，呈現在畫面上撥放它
URL我們先用國外測試的RTSP網址：rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mov，用VNC稍作測試可以播放後，就把它放進來opencv套件吧
```python
from cv2 import cv2

video = cv2.VideoCapture('rtsp://wowzaec2demo.streamlock.net/vod/mp4:BigBuckBunny_115k.mov')
```
這樣子相當於已經準備好跟RTSP的連線了，再來我們使用read()來取每一格frame，那他回傳的是一個陣列，第一個回傳狀態，第二個才是frame，所以我們用(status,frame)去接他，並用while去接每一格frame，while跳出條件當然是status囉，也就是當RTSP串流播放完畢跳出迴圈，另外，我會習慣再加上手動停止的flag，所以看起來會是這樣...
```python
(status, frame) = video.read()
while self.__active and status:
    (status, frame) = video.read()
```
取到frame後，會處理一個色彩空間轉換的步驟，把BGR轉換成我們常見的RGBA，讓他生成pixel陣列後，再透過PIL套件轉成Image物件
```python
imgArray = cv2.cvtColor(frame, cv2.COLOR_BGR2RGBA)
image = Image.fromarray(imgArray)
photo = ImageTk.PhotoImage(image=image)
```
最後，把RTSP影像放在tkinter的Label widget做呈現囉
```python
# 建立承載RTSP影像串流的容器物件
window = tk.Label(bg="black", image=photo, width=320, height=240)
# avoid garbage collection(避免資源被回收)(把圖片註冊到物件中，並自訂屬性，避免被回收)
window.rtspImage = photo
# 放置在畫面上
window.place(x=0, y=0, anchor='nw')
```
最後的最後，非常重要的一環，資源釋放，一定要切斷跟RTSP的連線，否則容易占用記憶體空間
```python
video.release()
```

# 我到底看了什麼? 錄下來吧
到這邊，客戶又給了一個新需求，要再檢視攝影機RTSP串流的時候，同步錄影下來，還好我們有OpenCV大神加持，沒問題，馬上錄；使用VideoWriter()提供的寫入影像功能就可以達成目的了
```python
record = cv2.VideoWriter([檔案路徑名稱], [影片編碼], [FPS], ([影片寬度], [影片高度]))
```
但是筆者在實做過程中，編碼如果跟來源不一樣，都沒有辦法成功錄進去檔案裡，於是再多做了一些處理，就是取得來源RTSP串流的編碼、FPS與寬高
```python
# 取得來源RTSP影像的編碼
def getSourceFourcc(sourceVideo):
    code = sourceVideo.get(cv2.CAP_PROP_FOURCC)
    code = "".join([chr((int(code) >> 8 * i) & 0xFF) for i in range(4)])
    return code

recordForucc = cv2.VideoWriter_fourcc(*getSourceFourcc(video))
recordFPS = int(video.get(cv2.CAP_PROP_FPS))
recordWidth = int(video.get(cv2.CAP_PROP_FRAME_WIDTH))
recordHeight = int(video.get(cv2.CAP_PROP_FRAME_HEIGHT))
# 建立錄影實體物件
record = cv2.VideoWriter('CameraRecord.avi', recordForucc, recordFPS, (recordWidth, recordHeight))
```
接著，在每次成功取得frame後，就write到檔案中，這個動作會根據設定的FPS來寫到影片檔案中
```python
record.write(frame)  # 錄製影片到檔案
```
最後最後，很重要，要記得釋放，影片檔案才不會一直被占用住喔
```python
record.release()
```