# 前言
在安裝tensorflow開發環境的時候，因為存在許多相依套件，為了避免影響到正式系統環境的套件版本，故使用docker容器，來把開發環境包在一個容器內執行，以下遵循官網安裝步驟，step-by-step做一個小記錄

# 移除舊版本的docker
舊版本docker名字是docker、docker.io或docker-engine等，這些都確認先移除乾淨
```
> sudo apt-get remove docker docker-engine docker.io containerd runc
```

# 設置套件來源儲存庫
由於OS中apt-get原生的docker套件已經很舊了，這邊我們要加入官網的套件儲存庫網址，而且因為透過HTTPS的關係，所以需要安裝額外套件支援
```
> sudo apt-get update
> sudo apt-get install apt-transport-https ca-certificates curl gnupg-agent software-properties-common
```
加入Docker的GPG key
```
> curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -
```
驗證一下Key with the fingerprint
```
> sudo apt-key fingerprint 0EBFCD88
```
可以加入docker的儲存庫了
```
> sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
```

# 安裝docker engine
加入docker新來源儲存庫後，要先update一下再行安裝囉
```
> sudo apt-get update
> sudo apt-get install docker-ce docker-ce-cli containerd.io
```

# run一個hello world的test image，試試看有沒有安裝成功
```
> sudo docker run hello-world
```

# 參考資料
以上是針對debian 10安裝docker的過程中，官網中有提到如果是樹梅派的Raspbian的版本，就不能走這套安裝方式，需要額外使用script，這邊筆者以後回到樹莓派操作的時候再試試看
https://docs.docker.com/engine/install/debian/#install-using-the-convenience-script